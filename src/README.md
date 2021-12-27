# oauth-apigateway-python-demo
## 操作步骤
**步骤一：认证服务器和授权服务器的搭建（以flask demo为例）**

1.使用该模板创建云函数。 本Demo为使用Flask构建的应用，包含四个API：

`/login`：用于登陆，跳转到登陆页面进行用户名密码认证

`/code`：用于验证用户名密码，颁发授权码

`/generate`：用于验证授权码，颁发token

`/verify`：用于校验token是否有效

**步骤二：配置API网关的授权码API**
1. 创建API网关服务，创建后端为SCF类型的API。
- 前端配置：前端路径/code，增加两个Query参数:username和password。
- 后端配置：后端类型为云函数SCF，云函数选择第一步生成的云函数。

**步骤三：配置API网关的授权API**
1. 生成一对JWK（JSON Web 密钥），用于token生成与验证的私钥与公钥， 私钥用于授权服务签发JWT，公钥配置到API网关中对请求验签，目前API网关支持的密钥对的加密算法为RSA SHA256，密钥对的加密的位数为2048。
可选择使用Flask应用中produce_key.py生成，或使用其他工具生成。
2. 在已创建的API网关服务中，创建授权 API。
    - 前端配置：前端路径/generate，鉴权类型选择 OAuth2.0，OAuth 模式选择授权 API。增加一个Query参数:code
    - 后端配置：后端类型为SCF云函数，函数为Flask应用中创建的云函数。token 位置选择 Header，公钥第一步生成的公钥，创建完成后单击【完成】。

**步骤四：配置API网关的业务API**
1. 在授权 API 的服务中，创建业务 API。
- 前端配置：鉴权类型选择 OAuth2.0，OAuth 模式选择业务 API，关联授权 API 选择刚刚创建的授权API。
- 后端配置时，后端类型选择 Mock 类型，返回 hello world。

**步骤五：验证**
1. 使用curl访问授权码API，请求时增加username和password参数，获取授权码。
```
GET /code?
    username=apigw
    &password=apigwpsw HTTP/1.1
  Host: service-xxxx.gz.apigw.tencentcs.com
```
```
HTTP/1.1 200 OK
apigwcode
```

2. 使用curl访问授权API，请求时增加code参数，填入第一步获取的授权码，获取Token。
```
GET /generate?
    code=apigwcode HTTP/1.1
  Host: service-xxxx.gz.apigw.tencentcs.com
```
```
HTTP/1.1 200 OK
eyJhbGciOixxxxxxIsInR5cCI6IkpXVCJ9.eyJleHAiOjxxxxxxwOCwid3VwIjo5MH0.UV1hOHE9k1xxxxx83eJig
```

3. 将token放在Authorization Header中请求业务API。
```
GET /work HTTP/1.1  
  Host: service-xxxx.gz.apigw.tencentcs.com
  Authorization: Bearer id_token="eyJhbGciOixxxxxxIsInR5cCI6IkpXVCJ9.eyJleHAiOjxxxxxxwOCwid3VwIjo5MH0.UV1hOHE9k1xxxxx83eJig"'
```
```
HTTP/1.1 200 OK
hello
```

在本Demo中，使用了授权码API，授权API和业务API三个API，实际使用中可以根据业务本身情况定义授权码生成与Token生成的逻辑。
