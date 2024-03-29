[OAuth2.0.pdf](file:///D:/零声Linux/openresty/OAuth2.0.pdf)

[(107条消息) 在Kong网关中使用OAuth2认证_kong oauth2_nklinsirui的博客-CSDN博客](https://blog.csdn.net/nklinsirui/article/details/118972961)

```
用法：

1、给service添加oauth2的插件（默认指定全部用户），注意要想多个service公用一个认证的token  需要把 config.global_credentials 设置为true，所有oauth2的插件中此参数是true的service，token都可共用。

2、新建一个用户consumer。

3、给之前创建的用户添加一个oauth2的认证方式（相当于创建了一个应用），需要一个或者多个回调地址（访问地址跟此回调地址没有关系；如果获取token的地址和回调地址不一样，则不能获取token）， 此时会自动生成client_id 和client_secret（每个client_id client_secret请求一次token，如果再次请求一次token，则上一次请求的token直接失效）

4、通过接口 POST x-www-form-urlencoded  https://192.168.8.60:8443/hello-oauth/oauth2/token 获取token （如果调用地址和回调地址不一样则不能获取token）hello-oauth是route配置的path

请求参数有：

client_id:tO7VJzTFNdAwxCusRpO1VqxZ3K5lLkUY

client_secret:CIj3fchCZNUbmVQr0ueG0URtAGW0LS0N

grant_type:client_credentials

注意点：通过/hello-oauth路由获取的token在，/hello-normal路由上不可用。（需改进，创建oauth2插件时设置此参数config.global_credentials = true）

 

oauth2参数详解：

name                                       自定义名称

service_id                                 指定绑定的service

enabled                                    是否启用插件 默认true

config.scopes                              用户访问范围 String（不确定写什么）

config.mandatory_scope                     请求token中是否必须要携带scopes  默认false 

config.token_expiration                    token有效时间，单位s，设置为0则用不失效，默认7200s

config.enable_authorization_code           oauth2的四种授权方式之一，code授权方式 默认false

config.enable_client_credentials           oauth2的四种授权方式之一，客户端授权方式 默认false

config.enable_implicit_grant               oauth2的四种授权方式之一，静默授权方式 默认false

config.enable_password_grant               oauth2的四种授权方式之一，密码授权方式 默认false 

config.auth_header_name                    发送请求时，请求头部携带token的参数 默认 authorization 

config.hide_credentials                    是否向service隐藏授权信息     默认false。

config.accept_http_if_already_terminated    

config.anonymous                            可以匿名访问的用户id（kong的用户），只有在认证失败的请款下才会用到此参数

config.global_credentials                   所有的oauth2插件是否可以共用一个token，默认false

config.refresh_token_ttl                    refresh_token的失效时间 默认 1209600s 
————————————————
版权声明：本文为CSDN博主「nobugnowork」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/dragonsmile/article/details/100042337
```

