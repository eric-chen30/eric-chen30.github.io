---
title: 前后端数据 AES 对称加密通信
date: 2024-03-28 09:51:00
tags:
---

为了确保前端通过 CryptoJS 传递给后端的 `bytes[]` 数据保持一致，你需要在前后端之间建立一个统一的数据传输格式。通常，这涉及到以下几个步骤：

###  前端数据加密

在前端，使用 CryptoJS 库对数据进行加密。例如，使用 AES 加密算法：

```javascript
// 引入 CryptoJS
import CryptoJS from 'crypto-js';

// 假设这是你的数据
let data = "这是需要加密的数据";

// 密钥和 IV，需要与后端保持一致
let key = CryptoJS.enc.Utf8.parse("你的16位密钥");
let iv = CryptoJS.enc.Utf8.parse("你的16位IV");

// 对数据进行加密
let encrypted = CryptoJS.AES.encrypt(data, key, {
  iv: iv,
  mode: CryptoJS.mode.CBC,
  padding: CryptoJS.pad.Pkcs7
});

// 将加密后的数据转换为 Base64 编码的字符串
let base64EncryptedData = encrypted.ciphertext.toString(CryptoJS.enc.Base64);

// 发送 Base64 字符串到后端
fetch('/api/endpoint', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ encryptedData: base64EncryptedData })
});
```

###  后端接收并解密数据

后端接收到加密的 Base64 字符串后，需要将其解密回原始的字节数组。以下是使用 Java 语言的一个示例：

```java
import org.apache.commons.codec.binary.Base64;
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class AESUtil {
    private static final String KEY = "你的16位密钥"; // 与前端相同的密钥
    private static final String IV = "你的16位IV"; // 与前端相同的 IV

    public static byte[] decrypt(String base64EncryptedData) throws Exception {
        // Base64 解码
        byte[] encryptedBytes = Base64.getDecoder().decode(base64EncryptedData);

        // 创建密钥和 IV
        SecretKeySpec secretKeySpec = new SecretKeySpec(KEY.getBytes("UTF-8"), "AES");
        IvParameterSpec ivParameterSpec = new IvParameterSpec(IV.getBytes("UTF-8"));

        // 创建和初始化 Cipher 对象
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivParameterSpec);

        // 解密数据
        byte[] originalBytes = cipher.doFinal(encryptedBytes);

        return originalBytes;
    }
}
```

> 在这个例子中，后端使用 Apache Commons Codec 库来处理 Base64 编码，然后使用 Java 内置的 `Cipher` 类来解密数据。确保后端使用的密钥和 IV 与前端加密时使用的相同。

###  注意事项

- 密钥和 IV 必须在前后端之间安全地共享，不应该硬编码在客户端代码中。
- 确保前后端使用的加密算法、模式和填充方式完全一致。
- 所有的加密和解密操作都应该在 HTTPS 连接中进行，以防止数据在传输过程中被拦截。
- 考虑到安全性，不建议在前端进行敏感数据的加密操作。应该在用户输入数据后，将数据安全地传输到后端，然后在后端进行加密处理。
