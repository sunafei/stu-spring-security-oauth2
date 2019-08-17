## OAUTH2密码模式
  密码模式是用户直接将自己的用户名密码交给client，client用用户的用户名密码直接换取AccessToken
![password](passwordgrant.png)  
用户将认证密码发送给client  
client拿着用户的密码向授权服务器请求Access Token  
授权服务器将Access Token和Refresh Token发送给client  
这种模式十分简单，但是却意味着直接将用户敏感信息泄漏给了client，因此这就说明这种模式只能用于client是我们自己开发的情况下。因此密码模式一般用于我们自己开发的，第一方原生App或第一方单页面应用。
## 测试
```
# 请求获取access_token
curl -i -X POST \
   -H "Content-Type:application/x-www-form-urlencoded" \
   -H "Authorization:Basic Y2xpZW50aWQ6c2VjcmV0" \
   -d "grant_type=password" \
   -d "username=123456" \
   -d "password=123456" \
   -d "scope=all" \
 'http://localhost/oauth/token'
# 通过获取到的access_token请求用户信息
curl -i -X GET \
  -H "Authorization:bearer 80d2587b-d266-47bb-a1c3-775c04a3a6d8" \
  'http://localhost/user/me'
```
## REFUSH_TOKEN
```
# 根据refush_token换取最新的access_token值
curl -i -X POST \
   -H "Content-Type:application/x-www-form-urlencoded" \
   -H "Authorization:Basic Y2xpZW50aWQ6c2VjcmV0" \
   -d "grant_type=refresh_token" \
   -d "refresh_token=29779042-6492-4512-b410-1aaf42e6ab17" \
 'http://localhost/oauth/token'
```
refush_token在不同场景下的效果  
1. 调用时access_token，refresh_token均未过期  
access_token会变,而且expires延长,refresh_token根据设定的过期时间,没有失效则不变
2. 调用时access_token过期，refresh_token未过期  
access_token会变,而且expires延长,refresh_token根据设定的过期时间,没有失效则不变
3. 调用时refresh_token过期  
invalid_token
4. 总结  
refresh_token必须在过期之前调用才能换新的token  
只要refresh_token有效，就可以直接用它来换新的access_token(失效时间为配置文件中指定的值)
## token的存储
token可以有不同的存储方式 都是TokenStore接口的实现类,demo中体现了下面三种  
可以通过修改application.yml的参数security.oauth2.storeType切换
1. jdbc  
需要有jdbc连接 默认配置为mysql数据库 执行resources/init.sql创建存储token表结构
2. redis  
token信息存储在redis中,需要配置redis连接
3. jwt  
jwt产生的access_token中直接包含了用户的基本信息,可以通过网址[jsonwebtoken.io](https://www.jsonwebtoken.io/)解析tonken中存在的用户信息

## spring security oauth2 源码解析
[Spring Security Oauth2之架构源码解读](https://www.imooc.com/article/details/id/31057) 
## 注意
代码参考慕课网视频