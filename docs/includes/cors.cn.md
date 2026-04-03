## CORS

本节内容仅在跨域场景下需要（即你有多个机器人 API 运行在 `localhost:8081`、`localhost:8082` 等端口上），并且希望将它们合并到一个 FreqUI 实例中。

??? info "技术说明"
    所有基于 Web 的前端都受到 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)（跨源资源共享）的约束。
    由于对 Freqtrade API 的大多数请求都需要认证，正确的 CORS 策略对于避免安全问题至关重要。
    此外，标准规范不允许对带有凭证的请求使用 `*` CORS 策略，因此必须适当配置此设置。

用户可以通过 `CORS_origins` 配置项，允许来自不同源 URL 的访问请求到达机器人 API。
该配置项包含一个允许的 URL 列表，这些 URL 被允许消费机器人 API 的资源。

假设你的应用部署在 `https://frequi.freqtrade.io/home/` - 这意味着需要进行以下配置：

```jsonc
{
    //...
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": ["https://frequi.freqtrade.io"],
    //...
}
```

在以下（非常常见的）情况中，FreqUI 可以通过 `http://localhost:8080/trade` 访问（这是你在导航到 FreqUI 时在导航栏中看到的地址）。
![freqUI url](assets/frequi_url.png)

此情况下的正确配置是 `http://localhost:8080` - 包含端口的 URL 主体部分。

```jsonc
{
    //...
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": ["http://localhost:8080"],
    //...
}
```

!!! Tip "末尾斜杠"
    `CORS_origins` 配置中不允许出现末尾斜杠（例如 `"http://localhots:8080/"`）。
    这样的配置不会生效，CORS 错误仍会存在。

!!! Note
    我们强烈建议同时将 `jwt_secret_key` 设置为一个随机的、只有你自己知道的字符串，以避免未经授权的访问。
