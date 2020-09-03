## 问题背景

工作中，在和其他服务供应商对接时，有时需要使用AES加密方式实现接口的联调。算法逻辑需要自己实现，现把流程整理如下:
另，基于这篇文章 [使用 PyCrypto 进行 AES/ECB/PKCS#5(7) 加密](http://likang.me/blog/2013/06/05/python-pycrypto-aes-ecb-pkcs-5 "Permanent Link to 使用 PyCrypto 进行 AES/ECB/PKCS#5(7) 加密")，PKC7填充方式等同于PKC5填充方式。

## 安装依赖

```pip3 install crypto```

## 代码实现

包括完整的代码及注解
```python

import base64
from Crypto.Cipher import AES

class AESCipher:

    def __init__(self, key):
        self.key = key[0:16] #只截取16位
        self.iv = "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0" # 16位字符，用来填充缺失内容，可固定值也可随机字符串，具体选择看需求。

    def __pad(self, text):
        """填充方式，加密内容必须为16字节的倍数，若不足则使用self.iv进行填充"""
        text_length = len(text)
        amount_to_pad = AES.block_size - (text_length % AES.block_size)
        if amount_to_pad == 0:
            amount_to_pad = AES.block_size
        pad = chr(amount_to_pad)
        return text + pad * amount_to_pad

    def __unpad(self, text):
        pad = ord(text[-1])
        return text[:-pad]

    def encrypt(self, raw):
        """加密"""
        raw = self.__pad(raw)
        cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
        return base64.b64encode(cipher.encrypt(raw))

    def decrypt(self, enc):
        """解密"""
        enc = base64.b64decode(enc)
        cipher = AES.new(self.key, AES.MODE_CBC, self.iv )
        return self.__unpad(cipher.decrypt(enc).decode("utf-8"))


if __name__ == '__main__':
    e = AESCipher('8ymWLWJzYA1MvLF8')
    secret_data = "6860795567181583<REQDATA></REQDATA>242BB99CE386F2B1EA19CCCF606D20E2"
    enc_str = e.encrypt(secret_data)
    print('enc_str: ' + enc_str.decode())
    dec_str = e.decrypt(enc_str)
    print('dec str: ' + dec_str)
```


```
输出:
>>> enc_str: gO80A2YMTzkYTzSe6MDjwYLq2X3Du6WXP5CEj1qdaX7b39Egp1Dxj+CGs+PqWkuRkKhPNTt8BPQZfRpi4zj+1UxXjYkO51sRLwgARTlZDKY=
>>> dec str: 6860795567181583<REQDATA></REQDATA>242BB99CE386F2B1EA19CCCF606D20E2
```