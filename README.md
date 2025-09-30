# OpenSSL 证书配置

## 示例

### 示例规划

| 类型 | 编号   | 开始时间           | 结束时间           | 一级 | 二级        | 三级                | 四级 |
| ---- | ------ | ------------------ | ------------------ | ---- | ----------- | ------------------- | ---- |
| 机构 | 0100E3 | 2015-10-29 15:10:29 | 3315-10-29 15:10:29 | ca   |             |                     |      |
| 机构 | 020101 | 2015-10-29 15:10:29 | 2225-10-29 15:10:29 |      | ca.vpn      |                     |      |
| 机构 | 03010101 | -                | -                  |      |             | ca.openvpn.vpn      |      |
| 机构 | 020201 | 2015-10-29 15:10:29 | 2225-10-29 15:10:29 |      | ca.internet |                     |      |
| 机构 | 03020101 | -                | -                  |      |             | ca.service.internet |      |
| 机构 | 03020102 | -                | -                  |      |             | ca.personal.internet |      |
| 机构 | 020401 | -                  | -                  |      | ca.test     |                     |      |

### 示例实验

#### 生成根证书并自签名

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 自签名。

```
[fsyyft@kvm-centos7-openssl pki]# cd /data/pki/
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca genkey ca.ppno.net 1024 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca req_ca
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca selfsign_ca
```

#### 生成并签名 Internet CA 证书

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 使用根证书进行签名。

```
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca genkey ca.internet.ppno.net 1024 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca req_ca_internet
[fsyyft@kvm-centos7-openssl pki]# export CRT_ENDDATE=22251029151029Z
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca sign_ca_internet
```

##### 生成并签名 Service CA 证书

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 使用 Internet CA 证书进行签名。
- 复制证书和私钥到 Internet 相关证书颁发目录。

```
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca genkey ca.service.internet.ppno.net 4096 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca req_ca_internet_service
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca sign_ca_internet_service
[fsyyft@kvm-centos7-openssl pki]# /usr/bin/cp ca_root/private/ca.service.internet.ppno.net.key.zip ca_internet/private/
[fsyyft@kvm-centos7-openssl pki]# /usr/bin/cp ca_root/certs/ca.service.internet.ppno.net.crt ca_internet/certs/
```

###### 生成并签名 test.ppno.net 服务端证书

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 使用 Service CA 证书进行签名。

```
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service genkey test.ppno.net 4096 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service req_test_ppno_net
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service sign_test_ppno_net
```

##### 生成并签名 Personal CA 证书

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 使用 Internet CA 证书进行签名。
- 复制证书和私钥到 Internet 相关证书颁发目录。

```
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca genkey ca.personal.internet.ppno.net 4096 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca req_ca_internet_personal
[fsyyft@kvm-centos7-openssl pki]# ca_root/bin/ca sign_ca_internet_personal
[fsyyft@kvm-centos7-openssl pki]# /usr/bin/cp ca_root/private/ca.personal.internet.ppno.net.key.zip ca_internet/private/
[fsyyft@kvm-centos7-openssl pki]# /usr/bin/cp ca_root/certs/ca.personal.internet.ppno.net.crt ca_internet/certs/
```

###### 生成并签名 test 个人证书

- 生成密钥（如果已经有，则将密钥放到 private 目录下）。
- 生成证书颁发请求。
- 使用 Personal CA 证书进行签名。

```
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service genkey test.personal 4096 123456 123456
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service req_personal_test
[fsyyft@kvm-centos7-openssl pki]# ca_internet/bin/service sign_personal_test
```

## 常用命令

### 查看私钥信息

```
openssl rsa -noout -text -in ca.ppno.net.key
```

### 查看请求文件信息

```
openssl req -noout -text -in ca.ppno.net.csr
```

### 查看证书信息

```
openssl x509 -noout -text -in ca.ppno.net.crt
```