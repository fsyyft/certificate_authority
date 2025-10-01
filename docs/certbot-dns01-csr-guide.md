# 使用 Certbot 基于 CSR 的 DNS-01 通配符证书申请与演练指南

本文档总结了本次在 Ubuntu 24.04 主机上，使用 snap 版 Certbot，基于已有 CSR（Certificate Signing Request）并通过 DNS-01 挑战申请通配符证书的完整流程。包含安装、测试演练（staging）、正式签发（production）、关键概念说明、常见问题、以及权威参考链接。读者可按本文逐步复现实验。

## 一、会话回顾与关键信息
- 环境与目标
  - 目标主机：Ubuntu 24.04.3 LTS（noble），内核 6.8.x。
  - 需求：尽量轻量；工具为 Certbot；使用自备 CSR（私钥由用户自持，不上服务器）；手动 DNS 验证；需要通配符证书；先在 Let’s Encrypt 测试环境演练，再切换生产环境。
- 已做操作
  - 安装 snap core 与 Certbot（snap 版），验证版本：`certbot 5.1.0`。
  - 校验 CSR 文件存在并解析 SAN：确认包含多条通配符与非通配符域名（例如 `*.pd.ppno.net`、`pd.ppno.net`、`*.local.pd...` 等）。
  - 在 CSR 所在目录使用 Let’s Encrypt 测试端点执行 `certonly --manual --preferred-challenges dns --csr <CSR>`，Certbot 成功进入交互，逐条打印出需要添加的 `_acme-challenge.<域名>` 的 TXT 记录，验证了命令与流程正确。
  - 注意：`--manual-public-ip-logging-ok` 在本机 Certbot 版本不可用，已移除；其余参数工作正常。

## 二、前提条件与准备
- 公共权威 DNS：DNS-01 挑战要求 CA（Let’s Encrypt）能从公网查询到你设置的 TXT 记录。仅内网可解析的域名会导致校验失败。
- 通配符限制：通配符（如 `*.example.com`）只能使用 DNS-01 验证，不能用 HTTP-01。
- CSR 准备：
  - CSR 中应包含所有要签发的域名（SAN），建议同时包含根域与通配符，例如 `-d example.com -d *.example.com`。
  - 使用 OpenSSL 查看 CSR：
    ```bash
    openssl req -in your.csr -noout -text | grep -A2 -i "Subject Alternative Name"
    ```
- 目录建议：使用一个专门目录保存本次签发产物，便于管理（使用 `--csr` 时，证书产物会保存在“当前工作目录”）。

## 三、安装 Certbot（snap 版）
适用于 Ubuntu 24.04，其他发行版可参考官方文档。

```bash
sudo snap install core
sudo snap install --classic certbot
# 可选：创建软链便于 PATH 调用
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot 2>/dev/null || true

certbot --version
# 期望输出：certbot 5.x.x
```

说明：在 Ubuntu 上通过 snap 安装 certbot 为官方推荐方式之一，版本更新及时，兼容性较好。

## 四、测试环境演练（Let’s Encrypt Staging）
使用 Let’s Encrypt 的测试端点进行“手动 DNS-01 + CSR”演练，不消耗正式额度，也不会签发可用证书，仅用于验证流程。

1) 切到 CSR 所在目录或新建演练目录
```bash
mkdir -p /path/to/issue-staging-YYYYMMDD
cd /path/to/issue-staging-YYYYMMDD
```

2) 执行演练命令（无邮箱，减少交互，演练推荐）
```bash
sudo /snap/bin/certbot certonly \
  --csr /absolute/path/to/your.csr \
  --manual --preferred-challenges dns \
  --server https://acme-staging-v02.api.letsencrypt.org/directory \
  --agree-tos --register-unsafely-without-email
```

3) Certbot 会逐条输出需要添加的 TXT 记录，例如：
```
Please deploy a DNS TXT record under the name:

_acme-challenge.sub.example.com.

with the following value:

<token_value>
```
- 为 CSR 中的每个域名添加对应 TXT 记录；某些情况会需要在同一主机名下设置多个 TXT 值（DNS 标准允许同名多值）。
- 添加完成后，等待生效（可把 TTL 临时设为 60 秒），使用 `dig` 验证：
  ```bash
  dig +short TXT _acme-challenge.sub.example.com
  ```
- 所有记录生效后，回到 Certbot 交互中按回车继续。若成功，将在“当前目录”生成类似 `0000_cert.pem`（证书）与 `0000_chain.pem`（链）。

提示：若演练失败，多半是 TXT 记录未生效或不是公共权威 DNS。请检查域名托管与记录值拼写。

## 五、正式环境签发（Production）
确认演练流程无误，即可改用生产端点签发正式可用证书。建议提供邮箱以接收到期通知。

```bash
mkdir -p /path/to/issue-prod-YYYYMMDD
cd /path/to/issue-prod-YYYYMMDD

sudo /snap/bin/certbot certonly \
  --csr /absolute/path/to/your.csr \
  --manual --preferred-challenges dns \
  --server https://acme-v02.api.letsencrypt.org/directory \
  -m you@example.com --agree-tos --no-eff-email
```

- 按提示为所有域名添加 TXT 记录，待生效后回车完成签发。
- 产物说明：
  - `0000_cert.pem`：最终证书（不含私钥）。
  - `0000_chain.pem`：中间证书链（可作为 fullchain 视你的服务需求而定）。
  - 私钥：不在本机生成，与你的 CSR 对应的私钥由你自行安全保管。

## 六、参数说明与要点
- `certonly`：只申请证书，不自动安装到 Web 服务器。
- `--csr <file>`：使用现有 CSR 申请证书（Certbot 不会生成私钥；产物写入当前目录）。
- `--manual`：手动挑战模式（打印挑战→你手动加 TXT）。
- `--preferred-challenges dns`：选择 DNS-01 验证（通配符必须）。
- `--server <url>`：ACME 目录端点；staging（测试）与 production（正式）不同。
- `-m/--agree-tos/--no-eff-email`：注册账户与同意条款；正式建议提供邮箱。
- 版本差异：`--manual-public-ip-logging-ok` 在某些版本不可用，省略即可。

## 七、关键知识点解释
- ACME 协议与挑战类型：
  - ACME（Automatic Certificate Management Environment）是自动化证书申请的协议（IETF RFC 8555）。
  - 常见挑战：HTTP-01、DNS-01、TLS-ALPN-01。
  - 通配符证书只能用 DNS-01（RFC 8555 也明确了挑战的使用方式）。
- DNS-01 验证原理：
  - CA 要求你在 `_acme-challenge.<域名>` 下添加 TXT 记录，值为挑战 token；CA 通过公共 DNS 查询验证你对此域名的控制权。
- 多值 TXT：
  - 当需要为同一 FQDN（例如 `_acme-challenge.example.com`）同时证明多个 SAN 时，可在同名 TXT 记录下添加多个值（DNS 标准允许）。
- 公共权威 DNS 必须：
  - 仅内网可解析的记录无法被 CA 验证，导致失败。
- 证书与链：
  - 部署时通常需要“证书 + 中间链”（有的服务称 `fullchain`），以便客户端完整构建信任链。

## 八、可选：一次性导出 TXT 清单（非交互汇总）
如果希望一次性拿到所有域名的 TXT 记录清单，可使用 `--manual-auth-hook` 脚本：

1) 创建脚本 `~/bin/dns-challenge-dump.sh`（示例）：
```bash
#!/usr/bin/env bash
set -euo pipefail
OUT="${CERTBOT_AUTH_OUTPUT:-/tmp/certbot-dns-challenges.txt}"
# CERTBOT_DOMAIN、CERTBOT_VALIDATION 由 Certbot 注入
{
  echo "_acme-challenge.${CERTBOT_DOMAIN}  TXT  \"${CERTBOT_VALIDATION}\""
} >> "$OUT"
```

2) 赋权并执行（staging 演练）：
```bash
chmod +x ~/bin/dns-challenge-dump.sh
export CERTBOT_AUTH_OUTPUT=/path/to/challenges.txt

sudo /snap/bin/certbot certonly \
  --csr /absolute/path/to/your.csr \
  --manual --preferred-challenges dns \
  --manual-auth-hook ~/bin/dns-challenge-dump.sh \
  --manual-cleanup-hook /bin/true \
  --server https://acme-staging-v02.api.letsencrypt.org/directory \
  --agree-tos --register-unsafely-without-email -n
```
- 说明：`-n` 设为非交互模式，Certbot 会依次调用 auth-hook，将所有挑战记录写入 `challenges.txt`，你可一次性去 DNS 添加；待全部生效后，再换回交互模式或以适当的脚本让 Certbot继续完成（进阶用法视需要调整）。

## 九、续签策略与注意
- 使用 `--csr` 的一次性签发不会进入 Certbot 的标准续签目录（`/etc/letsencrypt/live/...`），续期需要重复本流程。
- 建议：
  - 保存用于生成 CSR 的配置（SAN 列表），便于 90 天后快速再生成 CSR 并重复流程。
  - 若未来希望自动化，可考虑：
    - 使用 DNS 提供商 API + Certbot DNS 插件或 acme.sh，实现自动添加/清理 TXT 并定时续期。
- Let’s Encrypt 有效期：90 天。

## 十、常见问题排查
- TXT 记录查不到或值不匹配：
  - 使用 `dig +trace TXT _acme-challenge.example.com` 逐级排查权威 DNS。
  - 确认未加引号错误/空格/全角字符；确保记录发布在权威 DNS 上。
- 同名多值遗漏：
  - 有多个挑战值时，需在同名 TXT 下添加所有值。
- 内网域名：
  - 仅私网可见的域名无法通过 CA 公网校验。
- 频率限制（Rate Limits）：
  - 参考 Let’s Encrypt 官方文档（见下方链接），避免过多失败重试触发限制。
- CAA 记录限制：
  - 若你设置了 CAA 记录，需允许 Let’s Encrypt（`issue "letsencrypt.org"`）。

## 十一、参考资料与权威链接
- IETF RFC（协议标准）
  - RFC 8555: Automatic Certificate Management Environment (ACME)
    - https://www.rfc-editor.org/rfc/rfc8555
  - RFC 8737: ACME TLS ALPN Challenge
    - https://www.rfc-editor.org/rfc/rfc8737
  - RFC 1035: Domain names - implementation and specification（DNS 基础）
    - https://www.rfc-editor.org/rfc/rfc1035
  - RFC 6844 & RFC 8659: DNS Certification Authority Authorization (CAA)
    - https://www.rfc-editor.org/rfc/rfc6844
    - https://www.rfc-editor.org/rfc/rfc8659
- Let’s Encrypt 官方
  - 生产端点（ACME v2）：https://acme-v02.api.letsencrypt.org/directory
  - 测试端点（Staging）：https://acme-staging-v02.api.letsencrypt.org/directory
  - Rate Limits：https://letsencrypt.org/docs/rate-limits/
  - DNS-01 Challenge 说明：https://letsencrypt.org/docs/challenge-types/#dns-01-challenge
- Certbot 文档
  - 使用指南：https://eff-certbot.readthedocs.io/
  - Manual 插件：https://eff-certbot.readthedocs.io/en/stable/using.html#manual

## 十二、命令速查
- 查看 CSR 的 SAN：
  ```bash
  openssl req -in your.csr -noout -text | grep -A2 -i "Subject Alternative Name"
  ```
- 演练（staging）：
  ```bash
  sudo /snap/bin/certbot certonly \
    --csr /absolute/path/to/your.csr \
    --manual --preferred-challenges dns \
    --server https://acme-staging-v02.api.letsencrypt.org/directory \
    --agree-tos --register-unsafely-without-email
  ```
- 正式（production）：
  ```bash
  sudo /snap/bin/certbot certonly \
    --csr /absolute/path/to/your.csr \
    --manual --preferred-challenges dns \
    --server https://acme-v02.api.letsencrypt.org/directory \
    -m you@example.com --agree-tos --no-eff-email
  ```

## 附录：在未预装 snap 的系统上安装 snapd

Snap 依赖 systemd。若你的环境没有启用 systemd（例如部分容器或早期 WSL），snapd 无法工作。以下为常见发行版的安装与启用步骤。

通用验证与准备：
- 确认 systemd 运行：`pidof systemd`（返回非空表示运行）。
- 安装后建议重登或重启一次，使路径与服务完全生效。
- 验证命令：
  ```bash
  snap version
  snap list
  ```

### Ubuntu（最小化安装未带 snapd 的情况）
```bash
sudo apt update
sudo apt install -y snapd
sudo systemctl enable --now snapd.socket
# 兼容性：确保有 /snap 绑定（部分系统无需，但添加无害）
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true
# 可选：启用 AppArmor 配置（如存在）
sudo systemctl enable --now snapd.apparmor 2>/dev/null || true

# 验证
snap version
```

### Debian 11/12（bookworm/bullseye）
```bash
sudo apt update
sudo apt install -y snapd apparmor
sudo systemctl enable --now apparmor
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true

# 建议重启或重登，再验证
snap version
```

### RHEL / Rocky Linux / CentOS Stream 8/9
```bash
sudo dnf install -y epel-release
sudo dnf install -y snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true

snap version
```

### Fedora
```bash
sudo dnf install -y snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true

snap version
```

### Arch Linux
```bash
sudo pacman -Sy --needed snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true

snap version
```

### openSUSE Leap / Tumbleweed
```bash
sudo zypper install -y snapd
# openSUSE 上通常启用服务（或 socket），并启用 AppArmor（若提供）
sudo systemctl enable --now snapd
sudo systemctl enable --now snapd.apparmor 2>/dev/null || true
sudo ln -s /var/lib/snapd/snap /snap 2>/dev/null || true

snap version
```

### WSL2（Windows Subsystem for Linux）
- 要求在发行版中启用 systemd（Ubuntu 22.04/24.04 on WSL 支持）。编辑 `/etc/wsl.conf`：
  ```ini
  [boot]
  systemd=true
  ```
  退出所有 WSL 会话并在 PowerShell 中运行 `wsl --shutdown`，再重启 WSL。
- 然后按对应发行版（Ubuntu/Debian 等）的步骤安装 snapd。

### 安装后快速自检
```bash
# 安装核心包（可选但常用）
sudo snap install core
snap version

# 测试安装一个经典约束的应用示例（以 certbot 为例）
sudo snap install --classic certbot
/snap/bin/certbot --version
```

### 常见问题排查
- `/snap/bin` 不在 PATH：可创建软链或把 `/snap/bin` 加入 PATH（重登后生效）。
- 无法启动 snapd：确认 systemd 正常、`snapd.socket` 已启用；检查防火墙与代理设置。
- AppArmor 未启用（Debian/openSUSE）：安装并启用 `apparmor`，再启用 `snapd.apparmor`。
- 旧版本发行版：参考发行版文档选择对应的启用方式或考虑替代安装路径（例如使用 acme.sh 或容器化）。
