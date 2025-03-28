# 加解密

## RSA加解密

```csharp
using System;
using System.Text;
using System.Security.Cryptography;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        { 
            RSA rsa = new RSA();

            Console.WriteLine("PublicKey: " + rsa.PublicKey);
            Console.WriteLine(string.Empty);
            Console.WriteLine("PrivateKey: " + rsa.PrivateKey);
            Console.WriteLine(string.Empty);
            string text = rsa.Encrypt("Apple");
            Console.WriteLine("'Apple' RSA  加密 : " + text);
            Console.WriteLine(string.Empty);
            Console.WriteLine("RSA  解密 : " + rsa.Decrypt(text));
        }

        public class RSA
        {
            public RSA()
            {
                rsa = new RSACryptoServiceProvider(2048);
            }

            private RSACryptoServiceProvider rsa;

            /// <summary>PublicKey</summary>
            public String PublicKey { get { return rsa.ToXmlString(false); } }

            /// <summary>PrivateKey</summary>
            public String PrivateKey { get { return rsa.ToXmlString(true); } }

            /// <summary>
            /// 加密
            /// </summary>
            /// <param name="text"></param>
            /// <returns></returns>
            public String Encrypt(String text)
            {
                return Encrypt(text, this.PublicKey);
            }

            /// <summary>
            /// 加密
            /// </summary>
            /// <param name="text"></param>
            /// <param name="publickey"></param>
            /// <returns></returns>
            public String Encrypt(String text, String publickey)
            {
                using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider())
                {
                    rsa.FromXmlString(publickey);
                    byte[] plaintextBytes = Encoding.UTF8.GetBytes(text);
                    byte[] ciphertextBytes = rsa.Encrypt(plaintextBytes, false);
                    return Convert.ToBase64String(ciphertextBytes);
                }
            }

            /// <summary>
            /// 解密
            /// </summary>
            /// <param name="text"></param>
            /// <returns></returns>
            public String Decrypt(String text)
            {
                return Decrypt(text, this.PrivateKey);
            }

            /// <summary>
            /// 解密
            /// </summary>
            /// <param name="text"></param>
            /// <param name="privatekey"></param>
            /// <returns></returns>
            public String Decrypt(String text, String privatekey)
            {
                using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider())
                {
                    rsa.FromXmlString(privatekey);
                    byte[] ciphertextBytes = Convert.FromBase64String(text);
                    byte[] plaintextBytes = rsa.Decrypt(ciphertextBytes, false);
                    return Encoding.UTF8.GetString(plaintextBytes);
                }
            }
        }
    }
}
```

## AES加解密

```csharp
using System;
using System.IO;
using System.Security.Cryptography;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        { 
            AES aes = new AES();
            Console.WriteLine("Key: " + aes.Key);
            Console.WriteLine("IV: " + aes.IV);
            Console.WriteLine(string.Empty);
            string text = aes.Encrypt("Apple");
            Console.WriteLine("'Apple' AES  加密 : " + text);
            Console.WriteLine(string.Empty);
            Console.WriteLine("AES  解密 : " + aes.Decrypt(text));
        }

        public class AES
        {
            public AES()
            {
                aes = Aes.Create();
            }

            private Aes aes;

            /// <summary>Key</summary>
            public String Key { get { return Convert.ToBase64String(aes.Key); } }

            /// <summary>IV</summary>
            public String IV { get { return Convert.ToBase64String(aes.IV); } }

            /// <summary>
            /// 加密
            /// </summary>
            /// <param name="text"></param>
            /// <returns></returns>
            public String Encrypt(String text)
            {
                return Encrypt(text, this.Key, this.IV);
            }

            /// <summary>
            /// 加密
            /// </summary>
            /// <param name="text"></param>
            /// <param name="key"></param>
            /// <param name="iv"></param>
            /// <returns></returns>
            public String Encrypt(String text, String key, String iv)
            {
                using (Aes aesAlg = Aes.Create())
                {
                    // 設定 AES 加密參數
                    aesAlg.Key = Convert.FromBase64String(key);
                    aesAlg.IV = Convert.FromBase64String(iv);

                    // 建立加密器
                    ICryptoTransform encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);

                    // 使用 MemoryStream 儲存加密後的資料
                    using (MemoryStream msEncrypt = new MemoryStream())
                    {
                        using (CryptoStream csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                        {
                            using (StreamWriter swEncrypt = new StreamWriter(csEncrypt))
                            {
                                // 將明文寫入加密流
                                swEncrypt.Write(text);
                            }
                            return Convert.ToBase64String(msEncrypt.ToArray());
                        }
                    }
                }
            }

            /// <summary>
            /// 解密
            /// </summary>
            /// <param name="text"></param>
            /// <returns></returns>
            public String Decrypt(String text)
            {
                return Decrypt(text, this.Key, this.IV);
            }

            /// <summary>
            /// 解密
            /// </summary>
            /// <param name="text"></param>
            /// <param name="key"></param>
            /// <param name="iv"></param>
            /// <returns></returns>
            public String Decrypt(String text, String key, String iv)
            {
                using (Aes aesAlg = Aes.Create())
                {
                    // 設定 AES 解密參數
                    aesAlg.Key = Convert.FromBase64String(key);
                    aesAlg.IV = Convert.FromBase64String(iv);

                    // 建立解密器
                    ICryptoTransform decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);

                    // 將密文轉換為 byte 陣列
                    byte[] cipherBytes = Convert.FromBase64String(text);

                    // 使用 MemoryStream 儲存解密後的資料
                    using (MemoryStream msDecrypt = new MemoryStream(cipherBytes))
                    {
                        using (CryptoStream csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
                        {
                            using (StreamReader srDecrypt = new StreamReader(csDecrypt))
                            {
                                // 從解密流讀取明文
                                return srDecrypt.ReadToEnd();
                            }
                        }
                    }
                }
            }
        }
    }
}
```
