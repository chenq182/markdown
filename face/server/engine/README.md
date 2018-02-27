# 人脸识别 文档
---

## 1. API 文档
### 1.1 人脸比对
#### 1.1.1 请求说明
图片格式: `jpg`、`jpeg`、`bmp`

图片编码: `Base64`

请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/match`

请求实体:
```JavaScript
{
    "token": "some_token",
    "imgA": "some_Base64_string",
    "imgB": "some_Base64_string",
    "shaA": "sha1_of_imgA",
    "shaB": "sha1_of_imgB"
}
```
#### 1.1.2 返回说明
返回实体:
```JavaScript
{
    "result": "score",
    "msg": "some_float"
}
```
### 1.2 人脸查找
#### 1.2.1 请求说明
图片格式: `jpg`、`jpeg`、`bmp`

图片编码: `Base64`

请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/ident`

请求实体:
```JavaScript
{
    "token": "some_token",
    "img": "some_Base64_string",
    "sha": "sha1_of_img"
}
```
#### 1.2.2 返回说明
返回实体:
```JavaScript
{
    "result": "score",
    "msg": "[some_card_1,some_float_1[,some_card_2,some_float_2[,...]]]"
}
```
或
```JavaScript
{
    "result": "score",
    "msg": "[some_card_1[,some_card_2[,...]]]"
}
```
### 1.3 人脸图库
#### 1.3.1 图片入库 请求说明
图片格式: `jpg`、`jpeg`、`bmp`

图片编码: `Base64`

请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/store/create`

请求实体:
```JavaScript
{
    "token": "some_token",
    "card": "id_card",
    "img": "some_Base64_string",
    "sha": "sha1_of_img"
}
```
#### 1.3.2 图片入库 返回说明
返回实体:
```JavaScript
{
    "result": "ok",
    "msg": "some_string"
}
```
#### 1.3.3 人员注销 请求说明
请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/store/delete`

请求实体:
```JavaScript
{
    "token": "some_token",
    "card": "id_card"
}
//或
{
    "token": "some_token",
    "sha": "sha1_of_img"
}
```
#### 1.3.4 人员注销 返回说明
返回实体:
```JavaScript
{
    "result": "ok",
    "msg": "some_string"
}
```
### 1.4 错误码
若请求错误，服务器将返回的JSON文本包含以下参数：
- `result` 错误码
- `msg` 错误信息及描述

例如Token无效返回：
```JavaScript
{
    "result": "4001",
    "msg": "Invalid token:令牌无效"
}
```
__通用及业务错误码__
| 错误码 | 错误信息 | 描述 | 处理建议 |
|:-:|-|-|-|
| 3011 | Invalid sha1 | 该图片不存在 | 图片或已删除 |
| 3012 | Invalid card | 证件号不存在 | 人员或已删除 |
| 4001 | Invalid token | 令牌无效 | 请求新令牌 |
| 4011 | Invalid sha1 | 图片校验失败，传输失真 | 重试即可 |
| 4012 | Invalid card | 证件号无效 | 核查证件号，重试即可 |
| 4021 | Disk io error | 磁盘操作失败，IO过高 | 重试即可 |
| 4022 | Task interrupted | 任务中断 | 重试即可 |
| 4023 | Running overtime | 任务超时 | 重试即可 |
| 5001 | Internal error | 服务器内部错误 | 记录错误信息，服务器启动错误 |
| 5002 | Result parsing error | 结果解析错误 | 记录错误信息，重试即可 |
| 0000 | * | 未知错误 | 记录错误信息，查看原因 |
---

## 2. 鉴权认证机制
### 2.1 简介
平台支持 OAuth 2.0 协议的授权访问。关于 OAuth 2.0 协议规范，请参考[`这里`](http://oauth.net/2/)。
本章节的各个部分也将提供对应协议段落的超链接，以供参考。

授权访问协议遵循[`OAuth 2.0`](https://tools.ietf.org/html/rfc6749)；
图片浏览链接采用[`Bearer令牌`](https://tools.ietf.org/html/rfc6750)；
其它HTTP访问采用[`MAC令牌`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-02)。

令牌仅限于首个 **使用** 它的 **IP**；令牌的更新过程 **不会** 更换授权ip主机，**不会** 更换授权范围scope。
### 2.2 授权范围
当前授权范围包含以下业务：
<font face="Courier New">
| scope | 对应业务 | token_type |
|-|-|:-:|
| match | 人脸比对 | mac |
| ident | 人脸查找 | mac |
| store | 人脸图库 | mac |
| image | 图片浏览 | bearer |
</font>

### 2.3 获取 access_token
#### 2.3.1 请求说明
请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/token`

请求实体:

[`密码方式`](https://tools.ietf.org/html/rfc6749#section-4.3.2)
```JavaScript
{
    "grant_type": "password",
    "username": "some_user",
    "password": "some_password",
    "scope": "scope1[,scope2[,...]]"
}
```
[`可信IP方式`](https://tools.ietf.org/html/rfc6749#section-4.4.2)（仅限引擎本机）
```JavaScript
{
    "grant_type": "client_credentials",
    "scope": "scope1[,scope2[,...]]"
}
```
#### 2.3.2 返回说明
返回实体:

[`bearer令牌`](https://tools.ietf.org/html/rfc6749#section-5.1)
```JavaScript
{
    "access_token": "some_string",
    "token_type": "bearer",
    "expires_in": 3600,
    "refresh_token": "some_string",
    "scope": "scope1[,scope2[,...]]"
}
```
[`mac令牌`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-02#section-5.1)
```JavaScript
{
    "access_token": "some_string",
    "token_type": "mac",
    "expires_in": 2678400,
    "refresh_token": "some_string",
    "scope": "scope1[,scope2[,...]]",
    "mac_key": "some_string",
    "mac_algorithm": "hmac-sha-1"
}
```
### 2.4 更新 access_token
#### 2.4.1 请求说明
请求方法: `POST`

请求URL: `http://106.12.2.90/face01/rest/token`

请求实体: `参考`[`这里`](https://tools.ietf.org/html/rfc6749#section-6)
```JavaScript
{
    "grant_type": "refresh_token",
    "refresh_token": "some_refresh_token"
}
```
#### 2.4.2 返回说明
同前一个二级标题 `获取 access_token` 的 `返回说明`。
### 2.5 使用 access_token 访问授权业务
#### 2.5.1 使用bearer令牌
常见的bearer令牌只用于https协议访问，且有效时间较短，参考[`这里`](https://tools.ietf.org/html/rfc6750#section-5.3)。

本平台bearer令牌即其`access_token`字段值，使用范围如下：
- 内嵌于访问地址，且忽略`GET`请求参数
#### 2.5.2 使用mac令牌
本平台mac令牌支持两种访问方式：
- **"Authorization" Header 方式**，参考[`这里`](https://tools.ietf.org/html/rfc6749#section-7.1)，其中`Authorization`字段的值即为访问签名；
- **"token" 字段方式**，即访问请求的`json`实体包含`token`字段，其值即为访问签名。

仅当`json`实体不存在字段`token`（或值为空）时，以`"Authorization" Header 方式`认证。

不同方式下最终的访问签名保持一致，均以"`MAC`"作为前缀，参考[`示例`](https://tools.ietf.org/html/rfc6749#section-7.1)对应的访问签名如下：
```JavaScript
MAC id="h480djs93hd8", nonce="274312:dj83hs9s", mac="kDZvddkndxvhGRXZhvuDjEWhGeE="
```
如授权业务需要每个请求提供`bodyhash`字段，参考[`示例`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00#section-3.2)对应的访问签名如下：
```JavaScript
MAC id="jd93dh9dh39D", nonce="273156:di3hvdf8", bodyhash="k9kbtCIy0CkI3/FEfpS/oIDjk6k=", mac="W7bdMZbv9UWOTadASIQHagZyirA="
```
访问签名中`id`字段值取`access_token`的`access_token`字段值。

访问签名中`nonce`字段的取值参考[`这里`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00#section-3.1)，由令牌已生成的时长（单位：秒）和随机字符串组成，中间以英文冒号'`:`'间隔，以确保请求的单一性。

访问签名中`bodyhash`字段为可选项，依授权业务分别指定，若未指定，则不设此字段。

访问签名中`mac`字段值的生成步骤如下，参考[`这里`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00#section-3.3)：
- 生成规范化字符串`text`，参考[`这里`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00#section-3.3.1)。将`nonce`字段、请求方法、请求URI、访问域名或IP、访问端口、`bodyhash`字段、`ext`字段（当前为空串）依次连接，连接符为'`\n`'，示例如下：
```JavaScript
264095:7d8f3e4a\n
POST\n
/face01/rest/request\n
127.0.0.1\n
80\n
Lve95gjOVATpfV8EL5X4nxwjKHE=\n
\n
```
- 计算`mac = HMAC-SHA1 (key, text)`，其中`key`为访问签名`id`字段值，`text`为上述规范化字符串`text`。
### 2.6 错误码
若请求错误，服务器将返回JSON实体（仅含`error`字段），并设HTTP状态码；若`error`字段值为`invalid_client`，将附加Header。 参考[`这里`](https://tools.ietf.org/html/rfc6749#section-5.2)。

返回实体:
```JavaScript
{
    "error": "some_error_type"
}
```
__授权错误一览__
| error字段值 | 状态码 | 描述 |
|-|-|-|
| invalid_request | 400 | 请求字段不合法 |
| invalid_client | 401 | 无效终端、无认证方式、认证方式不支持 |
| invalid_grant | 400 | 令牌无效、令牌过期、令牌已删除、终端IP不一致、请求路径不一致 |
| unauthorized_client | 400 | 已认证，但无权限使用此grant_type |
| unsupported_grant_type | 400 | 不支持此grant_type |
| invalid_scope | 400 | 无效的授权范围、无权限访问此业务 |
__附加Header一览__
| error字段值 | token_type | 附加Header |
|-|:-:|-|
| invalid_client | [`mac`](https://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-00#section-4.1) | `WWW-Authenticate: MAC` |
| invalid_client | [`bearer`](https://tools.ietf.org/html/rfc6750#section-3.1) | `WWW-Authenticate: Bearer realm="example"` |

___

## 3. SDK 文档
### 3.1 JAVA语言
#### Face Java SDK下载链接
[`java.zip`](./sdk/java_v01.zip)
支持Java版本: __1.8+__
#### Face Java SDK目录结构
```Java
ac.trimps.wlw
  - rest
      - ident
          - IdentResponse
          - TIdent
      - match
          - TMatch
      - store
      - MyResponseJson
  - utils
      - DiskIO
      - MyByteChecksum
  - Main
```
#### 人脸比对样例
```Java
public class Sample {
    public static void main(String[] args) {
        Client client = ClientBuilder.newBuilder().build();
        WebTarget target = client.target("http://106.12.2.90/face01/rest/match");
        try {
            TMatch match = new TMatch(
                    "C:\\Users\\Public\\Pictures\\avatar.jpg",
                    "C:\\Users\\Public\\Pictures\\测试0.jpg",
                    "password");
            Response response = target.request()//.header()
                    .post(Entity.entity(match,MediaType.APPLICATION_JSON_TYPE));
            MyResponseJson value = response.readEntity(MyResponseJson.class);
            response.close();
            System.out.println("result: " + value.result);
            System.out.println("msg: " + value.msg);
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
        System.out.println("match ok!");
    }
}
```
#### 人脸查找样例
```Java
public class Sample {
    public static void main(String[] args) {
        Client client = ClientBuilder.newBuilder().build();
        WebTarget target = client.target("http://106.12.2.90/face01/rest/ident");
        try {
            TIdent ident = new TIdent(
                    "C:\\Users\\Public\\Pictures\\avatar.jpg",
                    "password");
            Response response = target.request()//.header()
                    .post(Entity.entity(ident,MediaType.APPLICATION_JSON_TYPE));
            MyResponseJson value = response.readEntity(MyResponseJson.class);
            response.close();
            System.out.println("result: " + value.result);
            System.out.println("msg: " + value.msg);
            IdentResponse identResponse = null;
            if (value.result.equals("distance"))
                identResponse = new IdentResponse(value.msg);
            if (identResponse != null) {
                System.out.println(identResponse.id[0]);
                System.out.println(identResponse.score[0]);
            }
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
        System.out.println("ident ok!");
    }
}
```
