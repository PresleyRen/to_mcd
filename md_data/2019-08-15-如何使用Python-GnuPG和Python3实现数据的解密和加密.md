---
layout:     post
title:      如何使用Python-GnuPG和Python3实现数据的解密和加密
subtitle:   
date:       2019-08-15
author:     P
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - python
---
## 介绍

[GnuPG包](https://www.gnupg.org/)提供用于生成和存储加密密钥的完整解决方案。它还允许您加密和签名数据和通信。

在本教程中，您将创建一系列使用Python 3和[python-gnupg](https://pythonhosted.org/python-gnupg/)模块的脚本。这些脚本允许您对多个文件进行签名和加密，并在运行脚本之前验证脚本的完整性。

## **准备**

在继续本教程之前，请完成以下条件：

- Ubuntu 16.04服务器，拥有sudo权限的非root用户。在本教程中，我们的用户将被命名为**sammy**。
- 确保您已安装Python 3和pip，[安装教程](https://cloud.tencent.com/developer/article/1014867?from=10680)详见链接。
- 创建GnuPG密钥对。

## **第1步 - 检索密钥对信息**

sudo pip3 install python-gnupg

完成准备中的GnuPG教程后，您将在主目录下的`.gnupg`存储密钥对。GnuPG使用用户名和电子邮件存储密钥，以帮助识别密钥对。在此示例中，我们的用户名是**sammy**，我们的电子邮件地址是`sammy@example.com`。

运行以下命令以获取可用密钥的列表：

```
$  gpg --list-keys
```

```
/home/sammy/.gnupg/pubring.gpg
-----------------------------
pub   2048R/4920B23F 2018-04-23
uid                  Sammy <sammy@example.com>
sub   2048R/50C06279 2018-04-23
```

记下`uid`输出行中显示的电子邮件地址。稍后您将需要它来识别您的密钥。

## **第2步 - 安装Python-GnuPG和签名文件**

使用您的密钥，您可以安装`python-gnupg`模块，该模块充当GnuPG的包装器，以实现GnuPG和Python 3之间的交互。使用此模块，您将能够创建执行以下操作的Python脚本：

- 为文件创建分离的签名，通过从文件中分离签名，为签名过程添加一层安全性。
- 加密文件。
- 解密文件。
- 验证分离的签名和脚本。

在继续测试这些文件上的脚本之前，您将首先创建脚本以及一些测试文件。

首先，让我们安装`python-gnupg`模块以及`fs`包，这将允许您打开，读取和编写测试文件。更新包索引，并使用`pip`命令安装这些包：

```
$ sudo apt-get update
$ sudo pip3 install python-gnupg fs
```

有了这些软件包，我们就可以继续创建脚本和测试文件。

要存储脚本和测试文件，请在主目录中创建一个名为`python-test`的文件夹：

```
$ cd ~/
$ mkdir python-test
```

移至此目录：

```
$ cd python-test/
```

接下来，让我们创建三个测试文件：

```
$ echo "This is the first test file" > test1.txt
$ echo "print('This test file is a Python script')" > test2.py
$ echo "This is the last test file" > test3.txt
```

要为我们的测试文件创建分离的签名，让我们创建一个名为`signdetach.py`的脚本，该脚本将定位执行它的目录中的所有文件。签名充当时间戳并证明文档的真实性。

分离的签名将存储在一个名为`signatures/`的新文件夹中，该文件夹将在脚本运行时创建。

用`nano`打开`signdetach.py`新文件：

```
nano signdetach.py
```

让我们首先导入脚本所需的所有模块。这些包括启用文件导航的`os`和`fs`包，以及`gnupg`：

> 
~/python-test/signdetach.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg
```

现在让我们设置GnuPG的加密密钥的目录。GnuPG默认存储其密钥在`.gnupg`上，所以让我们用我们的用户名配置它。请务必使用非root用户的名称替换**sammy**：

> 
~/python-test/signdetach.py


```
...
gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
```

接下来，让我们创建一个`home_fs`变量来将当前目录位置存储为文件对象。这将使脚本可以在执行它的目录中工作：

> 
~/python-test/signdetach.py


```
...
home_fs = open_fs(".")
```

到现在为止，您的脚本将如下所示：

> 
~/python-test/signdetach.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")
```

此配置块是您在本教程中使用时将在脚本中使用的基本模板。

接下来，添加代码以检查是否存在已命名`signatures/`的文件夹，如果该文件夹不存在则创建它：

> 
~/python-test/signdetach.py


```
...
if os.path.exists("signatures/"):
        print("Signatures directory already created")
else:
        home_fs.makedir(u"signatures")
        print("Created signatures directory")
```

创建一个空数组以存储文件名，然后扫描当前目录，将所有文件名附加到`files_dir`数组：

> 
~/python-test/signdetach.py


```
...
files_dir = []

files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)
```

脚本将要做的下一件事是为文件生成分离的签名。循环遍历`files_dir`数组将使用密钥环上的第一个私钥为每个文件创建签名。要访问私钥，您需要使用您设置的密码解锁。替换`"my passphrase"`为在准备项中生成密钥对时使用的密码：

> 
~/python-test/signdetach.py


```
...
for x in files_dir:
    with open(x, "rb") as f:
        stream = gpg.sign_file(f,passphrase="my passphrase",detach = True, output=files_dir[files_dir.index(x)]+".sig")
        os.rename(files_dir[files_dir.index(x)]+".sig", "signatures/"+files_dir[files_dir.index(x)]+".sig")
        print(x+" ", stream.status)
```

完成后，所有签名都将移动到该`signatures/`文件夹。您完成的脚本将如下所示：

> 
~/python-test/signdetach.py


```
 #!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

if os.path.exists("signatures/"):
    print("Signatures directory already created")
else:
    home_fs.makedir(u"signatures")
    print("Created signatures directory")

files_dir = []

files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)

for x in files_dir:
    with open(x, "rb") as f:
        stream = gpg.sign_file(f,passphrase="my passphrase",detach = True, output=files_dir[files_dir.index(x)]+".sig")
        os.rename(files_dir[files_dir.index(x)]+".sig", "signatures/"+files_dir[files_dir.index(x)]+".sig")
        print(x+" ", stream.status)
```

现在我们可以继续加密文件了。

## **第3步 - 加密文件**

在文件夹中执行加密脚本将导致该文件夹中的所有文件在名为`encrypted/`的新文件夹中被复制和加密。用于加密文件的公钥是与您在密钥对配置中指定的电子邮件相对应的公钥。

打开一个名为`encryptfiles.py`的新文件：

```
$ nano encryptfiles.py
```

首先，导入所有必需的模块，设置GnuPG的主目录，并创建当前的工作目录变量：

> 
~/python-test/encryptfiles.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")
```

接下来，让我们添加代码来检查当前目录是否已经有一个名为`encrypted/`的文件夹，如果它不存在则创建它：

> 
~/python-test/encryptfiles.py


```
...
if os.path.exists("encrypted/"):
        print("Encrypt directory exists")
else:
        home_fs.makedir(u"encrypted")
        print("Created encrypted directory")
```

在搜索要加密的文件之前，让我们创建一个空数组来存储文件名：

> 
~/python-test/encryptfiles.py


```
...
files_dir = []
```

接下来，创建一个循环来扫描文件夹中的文件并将它们附加到数组：

> 
~/python-test/encryptfiles.py


```
...
files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)
```

最后，让我们创建一个循环来加密文件夹中的所有文件。完成后，所有加密文件都将传输到该`encrypted/`文件夹。在此示例中`sammy\@example.com`是加密期间要使用的密钥的电子邮件ID。请务必将其替换为您在步骤1中记下的电子邮件地址：

> 
~/python-test/encryptfiles.py


```
...
for x in files_dir:
    with open(x, "rb") as f:
        status = gpg.encrypt_file(f,recipients=["sammy@example.com"],output= files_dir[files_dir.index(x)]+".gpg")
        print("ok: ", status.ok)
        print("status: ", status.status)
        print("stderr: ", status.stderr)
        os.rename(files_dir[files_dir.index(x)] + ".gpg", 'encrypted/' +files_dir[files_dir.index(x)] + ".gpg")
```

如果您的`.gnupg`文件夹中存储了多个密钥，并且希望使用特定的公钥或多个公钥进行加密，则需要在`recipients`里通过添加其他收件人或替换当前的收件人来修改阵列。

完成后，您的`encryptfiles.py`脚本文件将如下所示：

> 
~/python-test/encryptfiles.py


```
 #!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

if os.path.exists("encrypted/"):
    print("Encrypt directory exists")
else:
    home_fs.makedir(u"encrypted")
    print("Created encrypted directory")

files_dir = []

files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)

for x in files_dir:
    with open(x, "rb") as f:
        status = gpg.encrypt_file(f,recipients=["sammy@example.com"],output= files_dir[files_dir.index(x)]+".gpg")
        print("ok: ", status.ok)
        print("status: ", status.status)
        print("stderr: ", status.stderr)
        os.rename(files_dir[files_dir.index(x)] + ".gpg", "encrypted/" +files_dir[files_dir.index(x)] + ".gpg")
```

现在让我们看一下该过程的第二部分：一次解密和验证多个文件。

## **第4步 - 解密文件**

解密脚本与加密脚本的工作原理大致相同，只是它要在`encrypted/`目录中执行。启动时，`decryptfiles.py`将首先识别使用的公钥，然后在`.gnupg`文件夹中搜索相应的私钥以解密文件。解密的文件将存储在一个名为`decrypted/`的新文件夹中。

打开一个名为`decryptfiles.py`的新文件：

```
$ nano decryptfiles.py
```

首先插入配置设置：

> 
~/python-test/decryptfiles.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")
```

接下来，创建两个空数组以在脚本执行期间存储数据：

> 
~/python-test/decryptfiles.py


```
...
files_dir = []
files_dir_clean = []
```

这里的目标是让脚本将解密后的文件放入自己的文件夹中;否则，加密和解密的文件将混合，难以找到特定的解密文件。要解决此问题，您可以添加将扫描当前文件夹以查看decrypted/文件夹是否存在的代码，如果不存在，则创建该文件夹：

> 
~/python-test/decryptfiles.py


```
...
if os.path.exists("decrypted/"):
    print("Decrypted directory already exists")
else:
    home_fs.makedir(u"decrypted/")
    print("Created decrypted directory")
```

扫描文件夹并将所有文件名附加到`files_dir`数组：

> 
~/python-test/decryptfiles.py


```
...
files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)
```

所有加密文件都将以`.gpg`扩展名添加到其文件名中，表明它们已加密。但是，在解密它们时，我们希望在没有此扩展名的情况下保存它们，因为它们不再加密。

为此，循环遍历`files_dir`数组并从每个文件名中删除`.gpg`扩展名：

> 
~/python-test/decryptfiles.py


```
...
    for x in files_dir:
            length = len(x)
            endLoc = length - 4
            clean_file = x[0:endLoc]
            files_dir_clean.append(clean_file)
```

新的清理文件名存储在`file_dir_clean`数组中。

接下来，让我们遍历文件并解密它们。替换`"my passphrase"`为您的密码以解锁私钥：

```
...
for x in files_dir:
    with open(x, "rb") as f:
       status = gpg.decrypt_file(f, passphrase="my passphrase",output=files_dir_clean[files_dir.index(x)])
       print("ok: ", status.ok)
       print("status: ", status.status)
       print("stderr: ", status.stderr)
       os.rename(files_dir_clean[files_dir.index(x)], "decrypted/" + files_dir_clean[files_dir.index(x)])
```

完成后，您的脚本文件将如下所示：

> 
~/python-test/decryptfiles.py


```
 #!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

files_dir = []
files_dir_clean = []

if os.path.exists("decrypted/"):
    print("Decrypted directory already exists")
else:
    home_fs.makedir(u"decrypted/")
    print("Created decrypted directory")

files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)

for x in files_dir:
    length = len(x)
    endLoc = length - 4
    clean_file = x[0:endLoc]
    files_dir_clean.append(clean_file)

for x in files_dir:
    with open(x, "rb") as f:
       status = gpg.decrypt_file(f, passphrase="my passphrase",output=files_dir_clean[files_dir.index(x)])
       print("ok: ", status.ok)
       print("status: ", status.status)
       print("stderr: ", status.stderr)
       os.rename(files_dir_clean[files_dir.index(x)], "decrypted/" + files_dir_clean[files_dir.index(x)])
```

使用我们的解密脚本，我们可以继续验证多个文件的分离签名。

## **第5步 - 验证分离的签名**

要验证多个文件的分离数字签名，让我们编写一个`verifydetach.py`脚本。此脚本将搜索`signatures/`工作目录中的文件夹，并使用其签名验证每个文件。

打开一个名为`verifydetach.py`的新文件：

```
$ nano verifydetach.py
```

导入所有必需的库，设置工作和主目录，并创建空files_dir数组，如前面的示例所示：

> 
~/python-test/verifydetach.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

files_dir = []    
```

接下来，让我们扫描包含我们要验证的文件的文件夹。文件名将附加到空`files_dir`数组：

> 
~/python-test/verifydetach.py


```
...
files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
files_dir.append(f)
```

最后，让我们使用一个遍历`files_dir`数组的循环来验证每个文件是否有自己的分离签名，以搜索文件`signatures/`夹中每个文件的分离签名。当它找到分离的签名时，它将使用它验证文件。最后一行打印出每个文件验证的状态：

> 
~/python-test/verifydetach.py


```
...
for i in files_dir:
     with open("../../signatures/" + i + ".sig", "rb") as f:
         verify = gpg.verify_file(f, i)
         print(i + " ", verify.status)
```

完成后，您的脚本将如下所示：

> 
~/python-test/verifydetach.py


```
 #!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

files_dir = []   

files = [f for f in os.listdir(".") if os.path.isfile(f)]
for f in files:
    files_dir.append(f)

for i in files_dir:
    with open("../../signatures/" + i + ".sig", "rb") as f:
        verify = gpg.verify_file(f, i)
        print(i + " ", verify.status)
```

接下来，让我们来看看如何在服务器执行文件之前验证文件的签名。

## **第6步 - 验证文件**

最终脚本将在执行之前对其进行验证。从这个意义上讲，它类似于`verifydetach`，但它具有启动已验证脚本的附加功能。它的工作原理是将脚本的名称作为参数，然后验证该文件的签名。如果验证成功，脚本将向控制台发送消息并启动已验证的脚本。如果验证过程失败，脚本会将错误信息发布到控制台并中止文件执行。

创建一个名为`verifyfile.py`的新文件：

```
$ nano verifyfile.py
```

让我们首先导入必要的库并设置工作目录：

> 
~/python-test/verifyfile.py


```
#!/usr/bin/env python3

import os
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")
```

要使脚本正常工作，必须存储要验证和执行的文件名。为此，让我们创建一个名为`script_to_run`的新变量：

> 
~/python-test/verifyfile.py


```
...
script_to_run = str(sys.argv[1])
```

此变量获取第一个参数并将其存储在新创建的变量中。接下来，脚本将打开分离的签名文件，使用其签名验证`script_to_run`中的文件，然后在通过验证时执行：

> 
~/python-test/verifyfile.py


```
...
with open("../../signatures/" + script_to_run + ".sig", "rb") as f:
     verify = gpg.verify_file(f, script_to_run)
     print(script_to_run + " ", verify.status)
     if verify.status == "signature valid":
          print("Signature valid, launching script...")
          exec(open(script_to_run).read())
     else:
           print("Signature invalid or missing, ")
           print("aborting script execution")
```

完成的脚本将如下所示：

> 
~/python-test/verifyfile.py


```
 #!/usr/bin/env python3

import os
import sys
import fs
from fs import open_fs
import gnupg

gpg = gnupg.GPG(gnupghome="/home/sammy/.gnupg")
home_fs = open_fs(".")

script_to_run = str(sys.argv[1])

with open("../../signatures/" + script_to_run + ".sig", "rb") as f:
    verify = gpg.verify_file(f, script_to_run)
    print(script_to_run + " ", verify.status)
    if verify.status == "signature valid":
        print("Signature valid, launching script...")
        exec(open(script_to_run).read())
    else:
        print("Signature invalid or missing, ")
        print("aborting script execution")
```

我们已经完成了脚本的创建，但目前它们只能从当前文件夹中启动。在下一步中，我们将修改其权限以使其可全局访问。

## **第7步 - 使脚本在系统范围内可用**

为了便于让我们从系统上的任何目录或文件夹中执行脚本，并将它们放在我们的`$PATH`中。使用该`chmod`命令为非root用户提供文件所有者的可执行权限：

```
$ chmod +x *.py
```

现在要查找您的`PATH`置，请运行以下命令：

```
$ echo $PATH
```

```
-bash: /home/sammy/bin:/home/sammy/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

如果目录的权限允许，存储在您的`$PATH`中的文件可以从系统中的任何文件夹访问。您可以将脚本放在您的`$PATH`中的任何位置，但是现在让我们将脚本从`python-test/`目录移动到`/usr/local/bin/`。

请注意，我们在复制文件时删除扩展名`.py`。如果看一下我们创建的脚本的第一行，会看到`#!usr/bin/env python3`。这行称为shebang，它有助于操作系统识别执行代码时使用的bash解释器或环境。当我们执行脚本时，操作系统会注意到我们将Python指定为我们的环境，并将代码传递给Python执行。这意味着我们不再需要文件扩展名来帮助确定我们想要使用的环境：

```
$ sudo mv encryptfiles.py /usr/local/bin/encryptfiles
$ sudo mv decryptfiles.py /usr/local/bin/decryptfiles
$ sudo mv signdetach.py /usr/local/bin/signdetach
$ sudo mv verifyfile.py /usr/local/bin/verifyfile
$ sudo mv verifydetach.py /usr/local/bin/verifydetach
```

现在，只需运行脚本名称以及脚本可能从命令行获取的任何参数，就可以在系统中的任何位置执行脚本。在下一步中，我们将介绍如何使用这些脚本的一些示例。

## **第8步 - 测试脚本**

现在我们已经将脚本移动到了我们的`$PATH`，我们可以从服务器上的任何文件夹运行它们。

首先，使用`pwd`命令检查您是否仍在`python-test`目录中工作：

```
$ pwd
```

输出应该是：

```
/home/sammy/python-test
```

您在本教程前面创建了三个测试文件。运行该`ls -l`命令以列出文件夹中的文件：

```
$ ls -l
```

您应该看到文件`python-test`夹中存储了三个文件：

```
-rw-rw-r-- 1 sammy sammy 15 Apr 15 10:08 test1.txt
-rwxrwxr-x 1 sammy sammy 15 Apr 15 10:08 test2.py 
-rw-rw-r-- 1 sammy sammy 15 Apr 15 10:08 test3.txt
```

我们将测试这三个文件的脚本。您可以使用cat命令在加密前快速显示文件的内容，如下所示：

```
$ cat test1.txt
```

```
This is the first test file
```

让我们首先为所有文件创建分离签名。为此，请从当前文件夹中执行`signdetach`脚本：

```
$ signdetach
```

```
Created signatures directory
test2.py  signature created
test1.txt  signature created
test3.txt  signature created
```

请注意，在输出中，脚本检测到`signatures/`目录不存在，所以之后创建了它。然后它也创建了文件签名。

我们可以通过`ls -l`再次运行命令来确认：

```
$ ls -l
```

```
total 16
drwxrwxr-x 2 sammy sammy 4096 Apr 21 14:11 signatures
-rw-rw-r-- 1 sammy sammy 15 Apr 15 10:08 test1.txt
-rwxrwxr-x 1 sammy sammy 15 Apr 15 10:08 test2.py 
-rw-rw-r-- 1 sammy sammy 15 Apr 15 10:08 test3.txt
```

注意列表之间的新`signatures`目录。让我们列出这个文件夹的内容，并仔细查看其中一个签名。

要列出所有签名，请键入：

```
$ ls -l signatures/
```

```
total 12
-rw-rw-r-- 1 sammy sammy 473 Apr 21 14:11 test1.txt.sig
-rw-rw-r-- 1 sammy sammy 473 Apr 21 14:11 test2.py.sig
-rw-rw-r-- 1 sammy sammy 473 Apr 21 14:11 test3.txt.sig
```

`.sig`扩展可以识别分离的签名文件。同样，该cat命令可以显示这些签名的内容。我们来看看`test1.txt.sig`签名的内容：

```
$ cat signatures/test1.txt.sig
```

```
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1

iQEcBAABAgAGBQJa20aGAAoJENVtx+Y8cX3mMhMH+gOZsLJX3aEgUPZzDlKRWYec
AyrXEGp5yIABj7eoLDKGUxftwGt+c4HZud1iEUy8AhtW/Ea6eRlMFPTso2hb9+cw
/MyffTrWGpa0AGjNvf4wbxdq7TNpAlw4nmcwKpeYqkUu2fP3c18oZ3G3R3+P781w
GWori9FK3eTyVPs9E0dVgdo7S8G1pF/ECo8Cl4Mrj80rERAitQAMbSaN/dF0wUKu
okRZPJPVjd6GwqRRkXoqwh0vm4c+p3nAhFV+v7uK2BOUIJKPFbbn58vmmn+LVaBS
MFWSb+X85KwwftIezqCV/hqsMKAuhkvfIi+YQFCDXElJMtjPBxxuvZFjQFjEHe8=
=4NB5
-----END PGP SIGNATURE-----
```

此输出是`test1.txt`分离的签名。

有了签名，就可以继续加密我们的文件。为此，请执行以下`encryptfiles`脚本：

```
$ encryptfiles
```

```
Created encrypted directory
ok:  True
status:  encryption ok
stderr:  [GNUPG:] BEGIN_ENCRYPTION 2 9
[GNUPG:] END_ENCRYPTION

ok:  True
status:  encryption ok
stderr:  [GNUPG:] BEGIN_ENCRYPTION 2 9
[GNUPG:] END_ENCRYPTION

ok:  True
status:  encryption ok
stderr:  [GNUPG:] BEGIN_ENCRYPTION 2 9
[GNUPG:] END_ENCRYPTION
```

从输出中，注意脚本创建了`encrypted/`文件夹。另请注意，所有文件都已成功加密。再次运行`ls -l命令并注意目录中的新文件夹：

```
$ ls -l
```

```
total 20
drwxrwxr-x 2 sammy sammy 4096 Apr 21 14:42 encrypted
drwxrwxr-x 2 sammy sammy 4096 Apr 21 14:11 signatures
-rw-rw-r-- 1 sammy sammy   15 Apr 15 10:08 test1.txt
-rw-rw-r-- 1 sammy sammy   15 Apr 15 10:08 test2.py
-rw-rw-r-- 1 sammy sammy   15 Apr 15 10:08 test3.txt
```

让我们看看`test1.txt`现在是如何加密的：

```
$ cat encrypted/test1.txt.gpg
```

```
-----BEGIN PGP MESSAGE-----
Version: GnuPG v1

hQEMA9Vtx+Y8cX3mAQf9FijeaCOKFRUWOrwOkUw7efvr5uQbSnxxbE/Dkv0y0w8S
Y2IxQPv4xS6VrjhZQC6K2R968ZQDvd+XkStKfy6NJLsfKZM+vMIWiZmqJmKxY2OT
8MG/b9bnNCORRI8Nm9etScSYcRu4eqN7AeUdWOXAFX+mo7K00IdEQH+0Ivyc+P1d
53WBgWstt8jHY2cn1sLdoHh4m70O7v1rnkHOvrQW3AAsBbKzvdzxOa0/5IKGCOYF
yC8lEYfOihyEetsasx0aDDXqrMZVviH3KZ8vEiH2n7hDgC5imgJTx5kpC17xJZ4z
LyEiNPu7foWgVZyPzD2jGPvjW8GVIeMgB+jXsAfvEdJJAQqX6qcHbf1SPSRPJ2jU
GX5M/KhdQmBcO9Sih9IQthHDXpSbSVw/UejheVfaw4i1OX4aaOhNJlnPSUDtlcl4
AUoBjuBpQMp4RQ==
=xJST
-----END PGP MESSAGE-----
```

存储在原始文件中的句子已被转换为复杂的字符和数字系列。

现在文件已经过签名和加密，可以删除原件并从加密文件中恢复原始邮件。

要删除原件，请键入：

```
$ rm *.txt *.py
```

再次运行`ls -l`命令以确保已删除所有原始文件：

```
$ ls -l
```

```
total 8
drwxrwxr-x 2 sammy sammy 4096 Apr 21 14:42 encrypted
drwxrwxr-x 2 sammy sammy 4096 Apr 21 14:11 signatures
```

原始文件删除后，让我们解密并验证加密文件。切换到`encrypted`文件夹并列出所有文件：

```
cd encrypted/ && ls -l
```

```
total 12
-rw-rw-r-- 1 sammy sammy 551 Apr 21 14:42 test1.txt.gpg
-rw-rw-r-- 1 sammy sammy 551 Apr 21 14:42 test2.py.gpg
-rw-rw-r-- 1 sammy sammy 551 Apr 21 14:42 test3.txt.gpg
```

要解密文件，请从当前文件夹中运行`decryptfiles`脚本：

```
$ decryptfiles
```

```
OutputCreated decrypted directory

ok: True

status: decryption ok

stderr: [GNUPG:] ENC_TO D56DC7E63C717DE6 1 0

[GNUPG:] USERID_HINT D56DC7E63C717DE6 Autogenerated Key \<sammy\@example.com\>

[GNUPG:] NEED_PASSPHRASE D56DC7E63C717DE6 D56DC7E63C717DE6 1 0

[GNUPG:] GOOD_PASSPHRASE

gpg: encrypted with 2048-bit RSA key, ID 3C717DE6, created 2018-04-15

"Autogenerated Key \<sammy\@example.com\>"

[GNUPG:] BEGIN_DECRYPTION

[GNUPG:] DECRYPTION_INFO 2 9

[GNUPG:] PLAINTEXT 62 1524321773

[GNUPG:] PLAINTEXT_LENGTH 15

[GNUPG:] DECRYPTION_OKAY

[GNUPG:] GOODMDC

[GNUPG:] END_DECRYPTION

ok: True

status: decryption ok

stderr: [GNUPG:] ENC_TO D56DC7E63C717DE6 1 0

[GNUPG:] USERID_HINT D56DC7E63C717DE6 Autogenerated Key \<sammy\@example.com\>

[GNUPG:] NEED_PASSPHRASE D56DC7E63C717DE6 D56DC7E63C717DE6 1 0

[GNUPG:] GOOD_PASSPHRASE

gpg: encrypted with 2048-bit RSA key, ID 3C717DE6, created 2018-04-15

"Autogenerated Key \<sammy\@example.com\>"

[GNUPG:] BEGIN_DECRYPTION

[GNUPG:] DECRYPTION_INFO 2 9

[GNUPG:] PLAINTEXT 62 1524321773

[GNUPG:] PLAINTEXT_LENGTH 15

[GNUPG:] DECRYPTION_OKAY

[GNUPG:] GOODMDC

[GNUPG:] END_DECRYPTION

ok: True

status: decryption ok

stderr: [GNUPG:] ENC_TO D56DC7E63C717DE6 1 0

[GNUPG:] USERID_HINT D56DC7E63C717DE6 Autogenerated Key \<sammy\@example.com\>

[GNUPG:] NEED_PASSPHRASE D56DC7E63C717DE6 D56DC7E63C717DE6 1 0

[GNUPG:] GOOD_PASSPHRASE

gpg: encrypted with 2048-bit RSA key, ID 3C717DE6, created 2018-04-15

"Autogenerated Key \<sammy\@example.com\>"

[GNUPG:] BEGIN_DECRYPTION

[GNUPG:] DECRYPTION_INFO 2 9

[GNUPG:] PLAINTEXT 62 1524321773

[GNUPG:] PLAINTEXT_LENGTH 15

[GNUPG:] DECRYPTION_OKAY

[GNUPG:] GOODMDC

[GNUPG:] END_DECRYPTION
```

为每个文件返回的`status: decryption ok`脚本，意味着每个文件都被成功解密。

切换到新`decrypted/`文件夹并使用`cat`命令显示`test1.txt`的内容：

```
$ cd decrypted/ && cat test1.txt
```

```
This is the first test file
```

我们已经恢复了`test1.txt`中我们删除的文件中存储的消息。

接下来，让我们通过使用`verifydetach`脚本验证其签名来确认此消息确实是原始消息。

签名文件包含签名者的身份以及使用签名文档中的数据计算的哈希值。在验证期间，gpg将获取发送方的公钥并将其与散列算法一起使用以计算数据的哈希值。计算的散列值和签名中存储的值需要匹配才能使验证成功。

任何对原始文件，签名文件或发件人公钥的篡改都将导致哈希值发生变化，验证过程将失败。

从`decrypted`文件夹中运行脚本：

```
$ verifydetach
```

```
test2.py  signature valid
test1.txt  signature valid
test3.txt  signature valid
```

您可以从输出中看到所有文件都具有有效签名，这意味着在此过程中文档未被篡改。

现在让我们看一下在您签署文档后对文档进行更改时会发生什么。使用`nano`打开`test1.txt`文件：

```
nano test1.txt
```

现在将以下句子添加到文件中：

> 
~/python-test/encrypted/decrypted/test1.txt


```
This is the first test file
Let's add a sentence after signing the file
```

保存并关闭文件。

现在重新运行`verifydetach`脚本并注意输出的更改方式：

```
verifydetach
```

```
test2.py  signature valid
test1.txt  signature bad
test3.txt  signature valid
```

请注意，GnuPG在验证`test1.txt`时返回`signature bad`。这是因为我们在签名后对文件进行了更改。请记住，在验证过程中，`gpg`将签名文件中存储的哈希值与您从签名文档中计算的哈希值进行比较。我们对`test1.txt`文档所做的更改导致`gpg`计算出不同的哈希值。

对于我们的上一次测试，让我们在脚本执行之前使用`verifyfile`来验证脚本。此脚本可以看作是脚本`verifydetach`的扩展，但有以下区别：如果脚本通过验证过程，`verifyfile`将继续启动它。

该`test2.py`脚本在启动时会将字符串输出到控制台。让我们用它来演示`verifyfile`脚本的工作原理。

使用verifyfile运行test2.py脚本：

```
$ verifyfile test2.py
```

```
test2.py  signature valid
Signature valid, launching script...
The second test file is a Python script
```

从输出中可以看到脚本验证了文件的签名，根据该验证打印了适当的结果，然后启动了脚本。

让我们通过在文件中添加额外的代码行来测试验证过程。打开test2.py并插入以下代码行：

```
$ nano test2.py
```

> 
~/python-test/encrypted/decrypted/test2.py


```
print "The second test file is a Python script"
print "This line will cause the verification script to abort"
```

现在重新运行`verifyfile`脚本：

```
$ verifyfile test2.py
```

```
test2.py signature bad
Signature invalid, 
aborting script execution
```

脚本验证失败，导致脚本启动中止。

## **结论**

该python-gnupg模块允许在各种加密工具和Python之间进行集成。在某些情况下，例如查询或将数据存储到远程数据库服务器，快速加密或验证数据流完整性的能力至关重要。GnuPG密钥还可用于[加密Apple Mail的邮件](https://cloud.tencent.com/developer/article/1164229?from=10680)。

> 
参考文献：《How To Verify Code and Encrypt Data with Python-GnuPG and Python 3》

