=======================
openssl常用命令
=======================

常用操作
=================

#. 生成私钥::

    openssl genrsa -out cakey.pem 2048

#. 生成证书::

    openssl req -new -x509 -days 3650 -key cakey.pem -out cacert.pem

#. 生成证书签署请求::

    openssl req -new -key server.key -out server.csr -days 365

#. 根据请求签署证书::

    openssl x509 -req -in server.csr -CA /etc/pki/CA/cacert.pem -CAkey /etc/pki/CA/private/cakey.pem -CAcreateserial -out server.crt

流程
================

生成根证书: 1 -> 2

用根证书签名服务器证书: 1 -> 3 -> 4

服务器配置
=============

配置最佳实践: https://mozilla.github.io/server-side-tls/ssl-config-generator/

免费的证书认证机构
=======================
https://zh.wikipedia.org/wiki/Let%27s_Encrypt
