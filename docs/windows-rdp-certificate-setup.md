# Windows 10 远程桌面证书配置指南

## 引言

Windows 10 远程桌面（服务器端）的证书主要用于加密和验证远程连接的身份，通常指的是使用 SSL/TLS 证书来增强安全性。通过配置证书，可以确保远程桌面连接的安全性，防止中间人攻击，并提供身份验证。本文档详细介绍如何为 Windows 10 远程桌面会话主机配置证书，包括自签名证书和受信任 CA 证书的获取、导入以及相关配置步骤。

本文档基于证书管理系统生成的 `mstsc.ppno.net.pfx` 文件，强调完整证书链的导出和导入过程中的关键注意事项。

## 获取或生成证书

### 自签名证书

在 Windows 10 专业版或服务器版中，可以使用 PowerShell 生成自签名证书：

```powershell
# 以管理员身份运行 PowerShell
New-SelfSignedCertificate -DnsName "你的服务器名或IP" -CertStoreLocation "Cert:\LocalMachine\My"
```

此命令会在“个人”证书存储中生成一个自签名证书。

### 受信任 CA 证书

#### 购买证书

从受信任的证书颁发机构（如 Let's Encrypt、Comodo 或其他商业 CA）购买证书。

#### 使用证书管理系统生成证书

在本证书管理系统中，可以使用以下命令生成 RDP 证书：

```bash
# 在 ca_internet/bin/ 目录下运行
./internet export_mstsc_ppno.net
```

此命令会生成包含完整证书链的 `mstsc.ppno.net.pfx` 文件，并自动导出注册表修改脚本。

**重要提示**：导出证书时，必须包含完整的证书链（RDP 证书 + 服务 CA + 中间 CA + 根 CA），这是确保证书在 Windows 中正确验证的关键步骤。

## 在 Windows 10 中配置证书

### 打开 MMC 控制台

1. 按下 `Win + R` 键，输入 `mmc`，然后点击“确定”打开“管理控制台”。

![打开 MMC 控制台](images/mstsc/crt_manager.png)

### 添加证书管理单元

1. 在控制台菜单栏中，点击“文件” > “添加/删除管理单元”。
2. 在左侧选择“证书”，点击“添加”。
3. 选择“计算机账户”，点击“下一步”。
4. 确保选择“本地计算机”，然后点击“完成”和“确定”。

### 导入证书

1. 在证书管理单元的左侧导航栏中，展开“证书(本地计算机)” > “个人” > “证书”。
2. 右键点击“证书”，选择“所有任务” > “导入”。
3. 按照证书导入向导进行操作：
   - 选择 `mstsc.ppno.net.pfx` 文件。
   - 输入密码。
   - 选择“根据证书类型自动选择证书存储”。
   - **重要**：在导入过程中，确保勾选“标志此密钥为可导出的密钥”选项。
4. 完成导入后，验证证书是否出现在“个人”存储中。

![证书导入向导](images/mstsc/crt_manager_import.png)

### 设置远程桌面使用证书

#### 手动配置注册表

1. 按下 `Win + R`，输入 `regedit` 打开注册表编辑器。
2. 导航到以下路径：
   ```
   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp
   ```
3. 右键点击该路径，选择“新建” > “字符串值”，命名为 `SSLCertificateSHA1Hash`。
4. 双击该新值，将证书的指纹（SHA1 哈希值）复制并粘贴到“数值数据”字段中。
5. 点击“确定”保存更改。

![注册表编辑器配置 SSLCertificateSHA1Hash](images/mstsc/regedit_SSLCertificateSHA1Hash.png)

#### 使用自动生成的注册表脚本

如果使用证书管理系统生成的证书，可以直接运行导出的 `.reg` 文件：

```cmd
# 双击运行生成的注册表文件，或使用命令行
reg import mstsc.ppno.net.reg
```

此脚本会自动配置 `SSLCertificateSHA1Hash` 值，避免手动操作。

### 配置私钥权限

导入证书后，需要为“Network Service”用户授予私钥的读取权限：

1. 在 MMC 控制台中，右键点击导入的证书，选择“所有任务” > “管理私钥”。
2. 在“权限”对话框中，添加“Network Service”用户。
3. 授予“读取”权限。
4. 点击“应用”和“确定”。

![管理私钥 - 添加用户](images/mstsc/crt_manager_key_1.png)

![管理私钥 - 设置权限](images/mstsc/crt_manager_key_2.png)

或者使用 PowerShell 脚本自动配置权限：

```powershell
# 获取证书
$cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*mstsc.ppno.net*"}

# 配置权限
$keyPath = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
$keyPath = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\$keyPath"

# 授予 Network Service 读取权限
icacls $keyPath /grant "NT AUTHORITY\NETWORK SERVICE:(R)"
```

### 重新启动服务

配置完成后，重新启动远程桌面服务：

```powershell
# 重启远程桌面服务
Restart-Service termservice -Force
```

或者重新启动计算机以确保更改生效。

## 注意事项

- **权限要求**：确保用于远程桌面连接的用户（通常是 NETWORK SERVICE）对证书私钥拥有读取权限。导入后必须手动配置私钥权限。
- **证书类型**：如果使用自签名证书，客户端在连接时会收到安全警告，需要手动接受。对于生产环境，建议使用受信任 CA 签发的证书。

![连接时的安全警告](images/mstsc/connect_alert.png)
- **防火墙配置**：确保防火墙允许 RDP 端口（默认 3389）的入站连接。
- **证书链完整性**：导入的 PFX 文件必须包含完整的证书链，否则 Windows 可能无法正确验证证书。
- **备份**：在进行任何注册表修改前，建议备份注册表。
- **兼容性**：此配置适用于 Windows 10 专业版、企业版和服务器版。

## 故障排除

### 常见问题

1. **0x8009030D 错误**：通常是私钥权限问题。确保 NETWORK SERVICE 具有私钥读取权限。
2. **证书不被识别**：检查证书链是否完整，以及 SHA1 哈希是否正确配置在注册表中。
3. **连接失败**：验证防火墙设置和 RDP 服务状态。

### 诊断步骤

1. 检查证书导入：确认证书在“个人”存储中。
2. 验证注册表：检查 `SSLCertificateSHA1Hash` 值是否正确。
3. 检查权限：使用 `icacls` 命令验证私钥权限。
4. 查看事件日志：检查系统和应用程序事件日志中的 RDP 相关错误。

通过遵循本指南，您可以成功配置 Windows 10 远程桌面的证书，实现安全的远程连接。