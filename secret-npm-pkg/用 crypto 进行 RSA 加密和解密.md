```ts
import crypto from "crypto";

// 生成公钥和私钥
const { publicKey, privateKey } = crypto.generateKeyPairSync("rsa", {
  modulusLength: 2048,
  publicKeyEncoding: {
    type: "spki",
    format: "pem",
  },
  privateKeyEncoding: {
    type: "pkcs8",
    format: "pem",
  },
});

// 公钥加密
const encryptedData = crypto
  .publicEncrypt(
    {
      key: publicKey,
      padding: crypto.constants.RSA_PKCS1_PADDING,
    },
    Buffer.from(text)
  )
  .toString("base64");

// 私钥解密
const decryptedData = crypto
  .privateDecrypt(
    {
      key: privateKey,
      padding: crypto.constants.RSA_PKCS1_PADDING,
    },
    Buffer.from(encryptedData, "base64")
  )
  .toString();
```

注意，加密的数据有长度限制，经测试，在 modulusLength 为 `2048` 的情况下，可以加密的字节长度是 245 个字节（Byte），如果长度超过这个值，就会报错。

```txt
// ...
library: 'rsa routines',
reason: 'data too large for key size',
code: 'ERR_OSSL_RSA_DATA_TOO_LARGE_FOR_KEY_SIZE'
```

有个可行的解决方法是：用 AES 加密数据，然后用 RSA 加密 AES 密钥，这样操作后，加密过的 AES 密钥可以安全传输，然后通过 RSA 密钥解密后，使用 AES 密钥解密数据。
