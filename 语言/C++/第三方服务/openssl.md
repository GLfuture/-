Openssl：带有库openssl

直接使用命令：

```shell
sudo apt-get install openssl-dev
```

带有头文件

```
#include<openssl/sha.h>
#include<openssl/pem.h>
#include<openssl/bio.h>
#include<openssl/evp.h>
```

```
-lssl -lcrypto
```

