# VPN 证书配置

## OpenVPN 证书配置

### 服务端证书

| 编号 | IP | 开始时间 | 结束时间 | 四级 | 五级 |
| ---- | --- | -------- | -------- | ---- | ---- |
| 04010101000A    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | pi.server.openvpn.ppno.net | -    |
| 040101010035    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | qd.server.openvpn.ppno.net | -    |
| 0401010100D5    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | wd.server.openvpn.ppno.net | -    |

#### 操作步骤

1. 生成密钥（如果已经有，则将密钥放到 private 目录下）。
2. 创建证书颁发请求。
3. 使用 OpenVPN CA 签名。

```bash
[fsyyft@kvm-centos7-openssl pki]# cd /data/pki/
[fsyyft@kvm-centos7-openssl pki]# ca_vpn/bin/openvpn genkey 1024 123456 123456
```