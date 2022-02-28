---
title: Base64 编解码终结符问题
date: 2022-02-15 16:09:32
update: 2022-02-24 00:07:18
categories: 
  - [学习,问题]
tags:
  - Base64
  - 编解码
keywords: 
  - Base64
  - 编解码
cover: https://www.woolha.com/media/2020/12/base64-table.png
katex: true
---

# Base64 编解码终结符问题

## 1. 问题场景

工作时和甲方对接时，甲方测试对一个接口造了一个错误数据进行测试。其中一个参数是对原文进行采用 OPENSSL_PKCS1_PADDING 填充的 RSA 加密后，再进行 Base64 编码输出的数据进行传输，例如：若原文为 `password` 则加密后密文为：

```
D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA==
```

所造错误数据即在加密密文后多加了一个字符，但是依然可以解密成功。测试代码如下：

```java
import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;

/**
 * <p> RSA 加解密 </p>
 *
 * @author LiZijing
 * @date 2022/2/14
 */
public class RSA {
    private final static String CIPHER_ALGORITHM = "RSA/ECB/PKCS1Padding";
    private final static String PRIVATE_KEY = "MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDxxrtVFMNqwVrbjfYHzPbTt8Osd1cFiY1zAq/zcEdUmRoS+ujK8Qpw22mTWsAM4DqpGYr2qPKMHFmlZnj/3jG2ZLDKTMB1p2P06cgGWkl6M7TJp62aA9bNj1v/hK7wqEK1xXVkqROc+4Har4l5FZW/r6hAj2ireOdyPjx2++m11LGAJ5Cwprv6vGijNWLd4Qnd0dPFBd5mAGcFcnF9MvdcyME4xUHrOwmpLmALftLVWss/UZaUZxGiEdaBWdwAVaYb4yDnHTCvLbgVLRXQwChaf66YMASt975HlX3I3GijedQJmTfVhY94Z3qc5sqDakXvZ3YI6ZkMdZ4HXO0bS+C7AgMBAAECggEAFLbxL+3yfEAKt8rm7G4sK6GP+0PSSeAqJVNyncnd4qqnaD7lGRYjzd2OoxhgYfoILJrKpC1/cm+vYpNwBIQWAEmKOBrxVmM8Fiy9fYXYy8aIU8qw/gQcMEp7GF5W2rmf1ZEQaMpvqsCFtKXbgmtOBDlZkgZ3clGOiuQ4K/2TXYer9Sqk5LV9ctF6Ktpkq+a2PawRhmBlHpy34fvE7B9uQ39foRaS7m+YmYmjhZVQX7jICG82TSA/O9o2z4bwj/yIYYLBD15nllavWqNDj8wWOoAFPwYa2Lc93vZpJvpKJK5wFcsaz1fsrmMcrXntrt/kivfosmmt1qulwi+GCnxpuQKBgQD+gohiIhAJNFC+w5J+BQYR6qLlJqpaAGr/QsMRr91kmGgYWuQ5nNfapxVZgP2FfPZsS94liswGIr+xphqvrElo8p4MoAyE5deQX7qNzCidBNtxcii2Z7cFBai7u6Pzt/0qt34K1izzvFYg0c1weJ7hOcr3V9yhhMhAqmUI1CdpjQKBgQDzMR0PIL1acvHXirfXqFPrB6Pd9Qgm7rtzw8uN8j9xiJ9vK+xt+e08LaCdKtw3AOTFBaGMGpYmcBKallKjDb2Hs9F6H/ixCpEHUtlVImEyfobAlrdje6MhYXY2kUfoLZ1clB1tiReXwQBL3GV7vmbAoSpY7TcVJiIm+rRQVLVNZwKBgQCu0EwLU6g+GkAH999sTdkQf2DqEvfZoAXeVSYVxP1FtmVxrSSr6e5d0nwYoUAB64Z7dlUc5kwjPsT6qcQUvDskKdmjhF90/UZmdUp3US7oQ0jTkH0kZPLSMUPnxwfjRJJRP/4ERX5U4B0sp877nO5Md1zRLfluu/ysZh3FxatYlQKBgAPIALaqgKc2YFJEouUkheGCpeael7jbP2jmY3Tajmf6gtgcq7luCGVGJFgtQW1Ng0EY/FEMXMdOOMvUiIZmgUrp3djzRE+kZWriu+RZ+37ofrnh3goa8wdi146zpZWTl/3Hg8mfNxGx+4oybBWHeVuHZfwp/BBFHoTSoxkYqBUDAoGAWRE9Dv0mDHBhFLxQ2UTPJfE0w9bcMD5uN8nUIzzxlv+r/P2X20ImsEPNE97YGjUqZcps7Bpuo3WtMyKC5VPiNzH4U8E6b7hj3YGYP4b7SKYmh2MP4du+efyItwfOR7sptkMj6+vyCDRsZVBIWGfdsA4vrtLVb0kQJaD/v/WcjlI=";

    /**
     * 生成 PrivateKey
     * @param privateKeyStr privateKey 字符串
     * @return 私钥对象
     */
    public static PrivateKey generatePrivateKey(String privateKeyStr) {
        try {
            byte[] privateKeyBytes = Base64.decodeBase64(privateKeyStr);
            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            return keyFactory.generatePrivate(keySpec);
        } catch (Throwable t) {
            throw new RuntimeException("generate private key error", t);
        }
    }

    public static String decryptFromBase64(String ciphertext, Key pk) {
        String result;
        try {
            byte[] data = Base64.decodeBase64(ciphertext);
            Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, pk);
            byte[] decryptData = cipher.doFinal(data);
            result = new String(decryptData, StandardCharsets.UTF_8);
        } catch (Exception e) {
            e.printStackTrace();
            result = ciphertext;
        }
        return result;
    }

    public static void main(String[] args) {
        String res = "D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA==";
        String resAdd1 = "D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA==1";
        String resAdd3 = "D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA==123";
        String resAdd10 = "D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA==0123456789";
        String resAddF1 = "D9aRbiP4P9GIOvZOF1dhKqrAvihxVOfTwuCAKMiRJxRipLkUyRB2ES5B1mis6v2Cc7RYoG7q/9H3fnQopaHfhUJPLxAnX0GCksbXUhNfi+o6YMHoaXGnro+oVBKEMyHhh0eEgMVvBI/1dhb/UxHuUmyRfhkCcSf8R3jVDCmdXHXq203tm/Rgug7mLAGJvVB/D/dGcsHbu+HH956iPckRuD+p0e6KXGItV9BRQLRmuOPJmcMaDRgV4iN3lzCDFjbBJ2uysmjEroZNQMc00MwQbWWisB1i/blHJ1ruSe5opUtRMVfrRVekNPvRVxgjydGG4rxi7ZEFo+aPJvvZ+tmnyA1==";

        PrivateKey privateKey = generatePrivateKey(PRIVATE_KEY);

        System.out.println(res);
        System.out.println(decryptFromBase64(res, privateKey));
        System.out.println(resAdd1);
        System.out.println(decryptFromBase64(resAdd1, privateKey));
        System.out.println(resAdd3);
        System.out.println(decryptFromBase64(resAdd3, privateKey));
        System.out.println(resAdd10);
        System.out.println(decryptFromBase64(resAdd10, privateKey));
        System.out.println(resAddF1)
        System.out.println(decryptFromBase64(resAddF1, privateKey));
    }
}
```

测试结果如下：

![image-20220214222243317](https://s2.loli.net/2022/02/14/QX1Sd6ZpN2oncCM.png)

问题出现了，似乎在密文最后的 `==`  后面不管加多少位都不影响解密过程，但是在 `==` 前加就会报错。

对流程进行分析，RSA 加解密的流程似乎并没有在 `==` 后加了数据而受到影响，向公司的前辈请教，似乎是最后进行 Base64 编码导致的问题。 

## 2. Base64 的原理

此部分大量参考了阮一峰前辈的博客：[Base64笔记 - 阮一峰的网络日志 (ruanyifeng.com)](http://www.ruanyifeng.com/blog/2008/06/base64.html)

### 2.1 Base64 是什么

Base64（基底 64 ）是一种基于 64 个可打印字符来表示二进制数据的表示方法。由于 $\log_{2}{64}=6$​，所以每 6 个比特为一个单元，对应某个可打印字符。3 个字节相当于 24 个比特，对应于 4 个Base64单元，即 3 个字节可由 4 个可打印字符来表示。[^1]

[^1]: https://zh.wikipedia.org/wiki/Base64

所谓 Base64，就是说选出64个字符：小写字母 a-z、大写字母 A-Z、数字 0-9、符号 "+"、"/"（再加上作为垫字的 "="，实际上是65个字符）作为一个基本字符集。然后，其他所有符号都转换成这个字符集中的字符。[^2]

[^2]: http://www.ruanyifeng.com/blog/2008/06/base64.html

### 2.2 Base64 有什么用

1. 所有的二进制文件，都可以因此转化为可打印的文本编码，使用文本软件进行编辑；
2. 能够对文本进行简单的加密[^2]

### 2.3 Base64 怎么进行编解码

Base64 的编解码可以分为如下四步：

1. 将每 3 个字节作为一组，1 个字节有 8 个二进制位，共 24 个二进制位
2. 将 24 个二进制位分为 4 组，每组有 6 个二进制位
3. 在每组前面加 "00"，扩展为 32 个二进制位，共 4 个字节
4. 根据下表，得到扩展后的每个字节的对应符号，则为 Base64 的编码值
5. 解码过程反之

| 字节 | 符号 | 字节 | 符号 | 字节 | 符号 | 字节 | 符号 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | A    | 16   | Q    | 32   | g    | 48   | w    |
| 1    | B    | 17   | R    | 33   | h    | 49   | x    |
| 2    | C    | 18   | S    | 34   | i    | 50   | y    |
| 3    | D    | 19   | T    | 35   | j    | 51   | z    |
| 4    | E    | 20   | U    | 36   | k    | 52   | 0    |
| 5    | F    | 21   | V    | 37   | l    | 53   | 1    |
| 6    | G    | 22   | W    | 38   | m    | 54   | 2    |
| 7    | H    | 23   | X    | 39   | n    | 55   | 3    |
| 8    | I    | 24   | Y    | 40   | o    | 56   | 4    |
| 9    | J    | 25   | Z    | 41   | p    | 57   | 5    |
| 10   | K    | 26   | a    | 42   | q    | 58   | 6    |
| 11   | L    | 27   | b    | 43   | r    | 59   | 7    |
| 12   | M    | 28   | c    | 44   | s    | 60   | 8    |
| 13   | N    | 29   | d    | 45   | t    | 61   | 9    |
| 14   | O    | 30   | e    | 46   | u    | 62   | +    |
| 15   | P    | 31   | f    | 47   | v    | 63   | /    |

因为 Base64 将三个字节转化成四个字节，因此经过 Base64 编码后的文本要比原文本大三分之一左右。[^2]

**若最后不足三个字节怎么办？**

- **若最后有两个字节**
  - 两个字节共 16 个二进制位，按照上述规则可以分为：6+6+4
  - 对前两个 6 位二进制位组只需在前面补 "00" 即可，最后 4 位二进制位组除了在前面补 "00" 外，后面也要补上 "00" ，这样就可以得到 24 位二进制位。最后编码结果补 "=" 
- **若最后只有一个字节**
  - 一个字节共 8 个二进制位，按照上述规则可以分为：6+2
  - 最后 2 位二进制位组除了在前面补 "00" 外，后面也要补上 "0000"，这样就可以得到 16 位二进制位 最后编码结果补 "==" 

### 2.4 举个例子

例如，将文本 "Base64!" 进行 Base64 编码：

| 原文       | B        | a        | s        | e        | 6        | 4        | ！       |
| ---------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| ASCII 码值 | 66       | 97       | 115      | 101      | 54       | 52       | 33       |
| 二进制值   | 01000010 | 01100001 | 01110011 | 01100101 | 00110110 | 00110100 | 00100001 |

经过以上步骤得到文本 "Base64!" 的二进制值串：

`01000010 01100001 01110011 01100101 00110110 00110100 00100001`

三个字节为一组，进行转换：

`01000010 01100001 01110011 || 01100101 00110110 00110100 || 00100001`

`010000|10 0110|0001 01|110011 || 011001|01 0011|0110 00|110100 || 001000|01`

进行补 "0"，每组前面都有 "00"，下面省去了：

`010000|10 0110|0001 01|110011 || 011001|01 0011|0110 00|110100 || 001000|010000`

再转换回 ASCII 码值：

| 二进制值   | 010000 | 100110 | 000101 | 110011 | 011001 | 010011 | 011000 | 110100 | 001000 | 010000 |
| ---------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| ASCII 码值 | 16     | 38     | 5      | 51     | 25     | 19     | 24     | 52     | 8      | 16     |
| 符号       | Q      | m      | F      | z      | Z      | T      | Y      | z      | I      | Q      |

转换结果为：`QmFzZTYzIQ`，因为最后单独剩了一个字符，所以最后补 `==`，即最终结果为：`QmFzZTYzIQ==`

## 3. 总结

明显，在进行 RSA 加密后再用 Base64 编码输出的结果最后的 "=" 为终结符，终结符后不管加什么都不会再去进行解码，因此在 "=" 后面加字符制造的错误数据本身就是错误的...在等号前加就会改变原本的 Base64 编码结果，自然不能被 RSA 解密成功。

后续会再写写阮一峰前辈写的 RSA 加解密部分的总结😁。
