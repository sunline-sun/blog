### shiro 
- shiro 是一个开源的安全框架，可以提供身份验证、授权、密码学、会话管理，使用简单，虽然不如springsecurity功能强大，但是基本也够用了
- 功能
  - 验证用户，核实他们的身份
  - 控制用户的权限
  - 在任何环境下使用session api
  - 在身份验证，访问控制期间或在会话的生命周期，对事件做出反应
  - 单点登录功能
  
- 自己项目使用整个过程
  - 通过公共组件进行登录认证，通过注解的方式实现，登录接口加验签注解
  - 首先用户登录的时候阅文先会调用getaceessToken接口，通过租户的应用id和应用密钥获取accesstoken，后端把数据库密钥aes解密，验证传入的应用密钥是否正确，正确的话把应用id当做key存到redis里，生成一个uuid当value也就是accesstoken，再把这个value当做key，把租户信息存储进redis，返回accesstoken
  - 阅文通过应用密钥对登录的用户信息加密（hmacSha256Hex）生成签名，调用登录接口，公共组件拦截，后端也对用户信息相同方式加密，验证签名是否一致，不一致抛出异常，一致的话对用户信息jwt加密生成token返回给阅文，后面调用接口都通过这个token验证。
  - 创建shiroconfig配置类，增加jwt拦截器，并设置拦截范围，比如swagger接口，对外接口、测试类等都不需要认证
  - jwtfilter继承了BasicHttpAuthenticationFilter，在登录方法获取请求头中的token，调用subject.login(token)进行登录
  - 将subject实例委托给securityManage，通过securityManage.login方法开始真正的认证
  - securityManage会根据具体的realm来进行安全认证
  - realm为自定义的一个realm，继承了AuthorizingRealm类
  - myrealm类中doGetAuthenticationInfo方法中写了自己的验证方式，先解析请求头中的token，获取用户id，根据用户id去redis查询token，是否和请求头中的一致，不一致抛出异常，一致的话返回用户信息，AutorizingRealm会对用户和凭证进行验证
