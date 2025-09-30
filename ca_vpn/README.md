# VPN 证书配置

## OpenVPN 证书配置

### 服务端证书

| 编号 | IP | 开始时间 | 结束时间 | 四级
| ---- | --- | -------- | -------- | ---- |
| 04010101000A    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | pi.server.openvpn.ppno.net
| 040101010035    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | qd.server.openvpn.ppno.net
| 0401010100D5    | 172.27.xxx.1   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | wd.server.openvpn.ppno.net

#### 操作步骤

1. 生成密钥（如果已经有，则将密钥放到 private 目录下）。
2. 复制密钥和证书到 OpenVPN 相关目录。
3. 创建证书颁发请求；使用 OpenVPN CA 签名。

```bash
[fsyyft@kvm-centos7-openssl pki]# cd /data/pki/
[fsyyft@kvm-centos7-openssl pki]# ca_vpn/bin/openvpn genkey 1024 123456 123456
[fsyyft@kvm-centos7-openssl pki]# cp ca_root/private/ca.openvpn.vpn.ppno.net.key.zip ca_vpn/private
[fsyyft@kvm-centos7-openssl pki]# cp ca_root/certs/ca.openvpn.vpn.ppno.net.crt ca_vpn/certs
[fsyyft@kvm-centos7-openssl pki]# export CRT_ENDDATE=20331029151029Z
[fsyyft@kvm-centos7-openssl pki]# ca_vpn/bin/openvpn server qd
```

### 客户端证书

| 编号 | IP            | 开始时间           | 结束时间           | 四级 |
| ---- | ------------- | ------------------ | ------------------ | ---- |
| 040101010106 | 172.27.xxx.6   | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | teach.home.openvpn.ppno.net |
| 04010101010A | 172.27.xxx.10  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | hp.home.openvpn.ppno.net |
| 04010101010E | 172.27.xxx.14  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | test.home.openvpn.ppno.net |
| 040101010112 | 172.27.xxx.18  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | mac.home.openvpn.ppno.net |
| 040101010116 | 172.27.xxx.22  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | stock.home.openvpn.ppno.net |
| 04010101011A | 172.27.xxx.26  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | asus.home.openvpn.ppno.net |
| 04010101011E | 172.27.xxx.30  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | route.home.openvpn.ppno.net |
| 040101010122 | 172.27.xxx.34  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | temp.virtual.openvpn.ppno.net |
| 040101010126 | 172.27.xxx.38  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | internet.virtual.openvpn.ppno.net |
| 04010101012A | 172.27.xxx.42  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | research.virtual.openvpn.ppno.net |
| 04010101012E | 172.27.xxx.46  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | test.virtual.openvpn.ppno.net |
| 040101010132 | 172.27.xxx.50  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | vps.openvpn.ppno.net |
| 040101010136 | 172.27.xxx.54  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | test.work.openvpn.ppno.net |
| 04010101013A | 172.27.xxx.58  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | dev.work.openvpn.ppno.net |
| 04010101013E | 172.27.xxx.62  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | transfer.work.openvpn.ppno.net |
| 040101010142 | 172.27.xxx.66  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | beta.work.openvpn.ppno.net |
| 040101010146 | 172.27.xxx.70  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | ffs.mobile.openvpn.ppno.net |
| 04010101014A | 172.27.xxx.74  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | test.mobile.openvpn.ppno.net |
| 04010101014E | 172.27.xxx.78  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | cz.mobile.openvpn.ppno.net |
| 040101010152 | 172.27.xxx.82  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | ipad.mobile.openvpn.ppno.net |
| 040101010156 | 172.27.xxx.86  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | fsc.mobile.openvpn.ppno.net |
| 04010101015A | 172.27.xxx.90  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | iphone.mobile.openvpn.ppno.net |
| 04010101015E | 172.27.xxx.94  | 2015-10-29 15:10:29 | 2033-10-29 15:10:29 | csy.mobile.openvpn.ppno.net |
