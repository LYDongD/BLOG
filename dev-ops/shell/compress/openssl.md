## 秘钥工具 openssl

支持对称、非对称、摘要加密算法

参考[openssl](http://linux.51yip.com/search/openssl)

```
#base64
openssl base64 -in test.txt -out test_base.txt

#md5
openssl md5 test.txt > test2.txt

#sha1
openssl sha1 test.txt

#rsa, 私钥签名和公钥验证
openssl rsautl -sign -inkey rsaprivatekey.pem -in plain.txt -out signature.bin
openssl rsautl -verify -pubin -inkey rsapublickey.pem -in signature.bin -out plain

```
