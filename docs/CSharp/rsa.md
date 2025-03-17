# RSA加解密

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
