---
title: Web Api Owin+Oauth2.0（ClientCredentials）+Jwt Token权限认证控制
date: 2018-06-14 14:43:49
tags:
  - .NET
  - Oauth2
  - Owin
  - Jwt
categories:
  - .NET
---

# OAuth简介
OAuth简单说就是一种授权的协议，只要授权方和被授权方遵守这个协议去写代码提供服务，那双方就是实现了OAuth模式。

OAuth 2.0 四种授权模式：

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

下面的实列就是 客户端模式（client credentials）

# Jwt 简介
JSON Web Token（JWT）是一个开放式标准（RFC 7519），它定义了一种紧凑且自包含的方式，用于在各方之间以JSON对象安全传输信息。这些信息可以通过数字签名进行验证和信任。可以使用秘密（使用HMAC算法）或使用RSA的公钥/私钥对对JWT进行签名。

如上图所示引入对应得Nuget包。

​​![在这里插入图片描述](https://img-blog.csdn.net/20180614135736698)

在项目中创建 Startup.cs 文件，添加如下代码：

```csharp
/// <summary>
///     Startup
/// </summary>
public class Startup
{
    private readonly HttpConfiguration _httpConfig;

    /// <summary>
    ///     Initializes a new instance of the <see cref="Startup" /> class.
    /// </summary>
    public Startup()
    {
        _httpConfig = new HttpConfiguration();
    }

    /// <summary>
    ///     Configurations the specified application.
    /// </summary>
    /// <param name="app">The application.</param>
    public void Configuration(IAppBuilder app)
    {
        ConfigureOAuthTokenGeneration(app);
        ConfigureOAuthTokenConsumption(app);
    }
        //配置token生成
    private void ConfigureOAuthTokenGeneration(IAppBuilder app)
    {
        var oAuthServerOptions = new OAuthAuthorizationServerOptions
        {
            //TODO:For Dev enviroment only (on production should be AllowInsecureHttp = false)
            AllowInsecureHttp = true,
            TokenEndpointPath = new PathString("/oauth/token"),//获取token请求地址
            AccessTokenExpireTimeSpan = TimeSpan.FromDays(5),//token过期时间
            Provider = new SimpleOAuthProvider(),//token生成服务
            AccessTokenFormat = new SimpleJwtFormat()//token生成Jwt格式
        };
        app.UseOAuthAuthorizationServer(oAuthServerOptions);
        app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions());
    }
        //配置token使用
    private void ConfigureOAuthTokenConsumption(IAppBuilder app)
    {
        var issuer = ConfigurationManager.AppSettings["oauth:Issuer"];
        var audienceIds = ConfigurationManager.AppSettings["oauth:Audiences"];
        var audienceSecrets = ConfigurationManager.AppSettings["oauth:Secrets"];
        var allowedAudiences = audienceIds.Split(new[] {","}, StringSplitOptions.RemoveEmptyEntries);
        var base64Keys = audienceSecrets.Split(new[] {","}, StringSplitOptions.RemoveEmptyEntries);
        var keys = base64Keys.Select(s => TextEncodings.Base64Url.Decode(s)).ToList();
        app.UseJwtBearerAuthentication(new JwtBearerAuthenticationOptions
        {
            AuthenticationMode = AuthenticationMode.Active,
            AllowedAudiences = allowedAudiences,
            IssuerSecurityTokenProviders = new IIssuerSecurityTokenProvider[]
            {
                new SymmetricKeyIssuerSecurityTokenProvider(issuer, keys)
            },
            Provider = new SimpleOAuthBearerAuthenticationProvider("access_token")
        });
    }

    
}
```


SimpleOAuthProvider示例代码：


```csharp
public class SimpleOAuthProvider : OAuthAuthorizationServerProvider
    {
        public override  Task ValidateClientAuthentication(OAuthValidateClientAuthenticationContext context)
        {
           
            string clientId;
            string clientSecret;
            if (!context.TryGetBasicCredentials(out clientId, out clientSecret))
                context.TryGetFormCredentials(out clientId, out clientSecret);
            var authDbContext = new ExternalInterfaceBaseDbContext();
            ICustomerRepository customerRepository = new CustomerRepository(authDbContext);
            if (context.ClientId == null)
            {
                context.SetError("invalid_client", "The client_id is not set.");
                return Task.FromResult<object>(null); 
            }
            var secretKey = customerRepository.GetSecretKey(clientId);
            if (secretKey==null)
            {
                context.SetError("invalid_client", $"Invalid client_id '{context.ClientId}'.");
                return Task.FromResult<object>(null);
            }
            if (clientSecret != secretKey)
            {
                context.SetError("invalid_client", "Invalid client_Secret.");
                return Task.FromResult<object>(null);
            }

            context.Validated();
            return Task.FromResult<object>(null);
        }

        public override Task GrantClientCredentials(OAuthGrantClientCredentialsContext context)
        {
            context.OwinContext.Response.Headers["Access-Control-Allow-Origin"] = "*";
            var oAuthIdentity = new ClaimsIdentity(context.Options.AuthenticationType);
            oAuthIdentity.AddClaim(new Claim(ClaimTypes.Name, "External Interface"));
            var properties = new Dictionary<string, string> { { "audience", context.ClientId ?? string.Empty } };
            var authenticationProperties = new AuthenticationProperties(properties);
            var ticket = new AuthenticationTicket(oAuthIdentity, authenticationProperties);
            context.Validated(ticket);
            return base.GrantClientCredentials(context);
        }
    }
```

使用Oauth2.0 ClientCredentials 模式获取token，授予客户凭证 。注意：不同得模式重写。GrantResourceOwnerCredentials 内部可以调用外部服务，以进行对用户账户信息的验证。

SimpleJwtFormat 示例代码：

```csharp
public class SimpleJwtFormat : ISecureDataFormat<AuthenticationTicket>
    {
        private const string AudiencePropertyKey = "audience";
        private readonly string _issuer;
        private readonly string _audienceSecrets;

        public SimpleJwtFormat()
        {
            _issuer = ConfigurationManager.AppSettings["oauth:Issuer"];
            _audienceSecrets = ConfigurationManager.AppSettings["oauth:Secrets"];
        }

        public string Protect(AuthenticationTicket data)
        {
            if (data == null)
                throw new ArgumentNullException(nameof(data));
            var properties = data.Properties;
            var propertityDictionary = properties.Dictionary;
            var audienceId = propertityDictionary.ContainsKey(AudiencePropertyKey)
                ? propertityDictionary[AudiencePropertyKey]
                : null;
            if (string.IsNullOrWhiteSpace(audienceId))
                throw new InvalidOperationException("AuthenticationTicket.Properties does not include audience.");
            if (properties.IssuedUtc == null)
                throw new InvalidOperationException("AuthenticationTicket.Properties does not include issued.");
            if (properties.ExpiresUtc == null)
                throw new InvalidOperationException("AuthenticationTicket.Properties does not include expires.");
            var issued = properties.IssuedUtc.Value.UtcDateTime;
            var expires = properties.ExpiresUtc.Value.UtcDateTime;
            //TODO:
            //var authDbContext = new InstrumentDbContext();
            //var audienceRepository = new AudienceRepository(authDbContext);
            //var audience = audienceRepository.Get(audienceId);
            var decodedSecret = TextEncodings.Base64Url.Decode(_audienceSecrets);
            var signingCredentials = new HmacSigningCredentials(decodedSecret);
            var token = new JwtSecurityToken(_issuer, audienceId, data.Identity.Claims, issued, expires,
                signingCredentials);
            var handler = new JwtSecurityTokenHandler();
            var jwt = handler.WriteToken(token);
            return jwt;
        }
    }
```

根据授权Id生成Jwt Token返回。

SimpleOAuthBearerAuthenticationProvider 示例代码：

```csharp
public class SimpleOAuthBearerAuthenticationProvider : OAuthBearerAuthenticationProvider
    {
        private readonly string _accessTokenName;

        public SimpleOAuthBearerAuthenticationProvider(string accessTokenName)
        {
            _accessTokenName = accessTokenName;
        }

        public override Task RequestToken(OAuthRequestTokenContext context)
        {
            var token = context.Request.Query.Get(_accessTokenName);
            if (!string.IsNullOrEmpty(token))
                context.Token = token;
            return Task.FromResult<object>(null);
        }
    }
```

配置AccessToken使用。

到这里权限认证代码基本完成，最后就是在控制器或者方法上配置权限控制特性 [Authorize]。
```csharp
[Authorize]
[RoutePrefix("api/areas")]
public class AreaController : ApiController
```
这样Web Api Owin + Oauth2.0 + Jwt Token 的权限认证框架就已经搭好了。