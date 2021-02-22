
## 一、LINUX 下编译

使用 -d 代表编译debug版本， no-shared代表生成静态库

    ./config  no-shared  -d
    make
    make  install

## 二、Windows下编译

perl 建议选择开源的 Strawberry ， 不要选择商业版的 ActivePerl，要不然会碰到其他的小问题需要解决
编译静态库增加参数 no-shared  
执行nmake需要从开始菜单中选择 VS 文件夹中的控制台应用，我这里是Visual Studio 2015
32位选择 VS2015 x86 本机工具命令提示符
64位选择 VS2015 X64 本机工具命令提示符

### Windows下编译32位静态库
    perl Configure  VC-WIN32  no-shared
    nmake

### Windows下编译64位静态库
    perl Configure  VC-WIN64A  no-shared
    nmake




## 三、生成根证书
    1、生成私钥key
    $ gmssl.exe  ecparam -genkey -name sm2p256v1 -text -out Root.key -config  ./openssl.cnf
 
    2、生成签名请求
    $ gmssl.exe req -new -key Root.key -out Root.req -subj /C=CN/ST=ShangHai/L=SH/O=Root/OU=RootSign/CN=Root/emailAddress=Root@gmail.com -config ./openssl.cnf
 
    3、生成根证书
    $ gmssl.exe x509 -req -sm3 -days 3650 -in Root.req -signkey Root.key -out Root.crt 


## 四、生成客户端证书
    1、生成私钥key
    $ gmssl.exe  ecparam -genkey -name sm2p256v1 -text -out Client.key -config  ./openssl.cnf
 
    2、生成客户证书请求
    $ gmssl.exe req -new -key Client.key -out Client.req -subj /C=CN/ST=ShangHai/L=GZ/O=Client/OU=ClientSign/CN=Client/emailAddress=Client@gmail.com  -config  ./openssl.cnf
 
    3、签发证书
    $ gmssl.exe x509 -req -sm3 -days 3650 -in Client.req  -CA  Root.crt -CAkey Root.key -CAcreateserial  -out Client.crt

    4、证书验证
    $ gmssl.exe verify -CAfile Root.crt Client.crt

    5、证书转换成浏览器认识的格式.pfx
    $ gmssl.exe pkcs12 -export  -inkey Client.key -in Client.crt -out browser.pfx -passin  pass:xxx -passout pass:xxx
 
 
 
## 五、生成服务器证书
    1、生成私钥key
    $ gmssl.exe ecparam -genkey -name sm2p256v1 -text -out Server.key -config ./openssl.cnf

    2、证书请求
    $ gmssl.exe req -new -key Server.key -out Server.req -subj /C=CN/ST=ShangHai/L=GZ/O=Server/OU=ServerSign/CN=Server/emailAddress=Server@gmail.com -config ./openssl.cnf

    3、签发证书
    $ gmssl.exe x509 -req -sm3 -days 3650 -in Server.req -CA Root.crt -CAkey Root.key -CAcreateserial -out Server.crt

    4、证书验证
    $ gmssl.exe verify -CAfile Root.crt Server.crt

## 六、测试证书

	把生成的证书放到根目录中的apps目录，从控制台执行如下命令， 先执行服务器端，再执行客户端

    $ gmssl.exe s_client -debug -status -security_debug          -config "./openssl.cnf" -port "9999" -CAfile ".\\certs\\CA.crt" -cert ".\\certs\\ClientSign.crt"  -key ".\\certs\\ClientSign.key" -dcert ".\\certs\\ClientEnc.crt" -dkey ".\\certs\\ClientEnc.key" -gmtls
    $ gmssl.exe s_server -debug -status_verbose -security_debug  -config "./openssl.cnf" -port "9999" -CAfile ".\\certs\\CA.crt" -cert ".\\certs\\ServerSign.crt"  -key ".\\certs\\ServerSign.key" -dcert ".\\certs\\ServerEnc.crt" -dkey ".\\certs\\ServerEnc.key" -gmtls


