# FreqUI

Freqtrade 提供了一个内置的 Web 服务器，可以运行 [FreqUI](https://github.com/freqtrade/frequi)，即 freqtrade 的前端界面。

默认情况下，UI 会在安装过程中自动安装（脚本安装、Docker 安装）。
FreqUI 也可以通过 `freqtrade install-ui` 命令手动安装。
同样的命令也可以用来将 freqUI 更新到新版本。

一旦机器人以交易/模拟运行模式启动（使用 `freqtrade trade`），UI 将在配置的 API 端口下可用（默认为 `http://127.0.0.1:8080`）。

??? Note "想要为 freqUI 贡献代码？"
    开发者不应使用此方法，而应克隆 [freqUI 仓库](https://github.com/freqtrade/frequi)中描述的方法来获取 freqUI 的源代码。构建前端需要安装 node 环境。

!!! tip "freqUI 不是运行 freqtrade 的必要条件"
    freqUI 是 freqtrade 的可选组件，不是运行机器人的必需品。
    它是一个可用于监控和与机器人交互的前端界面——但 freqtrade 本身在没有它的情况下也能完美运行。

## 配置

FreqUI 没有自己的配置文件——但它假设 [rest-api](rest-api.md) 的配置已经正确设置。
请参考相应的文档页面来完成 freqUI 的设置。

## 界面

FreqUI 是一个现代化的响应式 Web 应用程序，可用于监控和与您的机器人交互。

FreqUI 提供浅色和深色两种主题。
主题可以通过页面顶部的醒目按钮轻松切换。
本页面截图的主题会根据所选的文档主题自动调整，因此要查看深色（或浅色）版本，请切换文档的主题。

### 登录

下面的截图显示了 freqUI 的登录界面。

![FreqUI - login](assets/frequi-login-CORS.png#only-dark)
![FreqUI - login](assets/frequi-login-CORS-light.png#only-light)

!!! Hint "CORS"
    此截图中显示的 CORS 错误是因为 UI 运行在与 API 不同的端口上，且 [CORS](#cors) 尚未正确配置。

### 交易视图

交易视图允许您可视化机器人正在进行的交易并与机器人交互。
在此页面上，您还可以通过启动和停止机器人来与之交互，并且——如果已配置——强制触发交易入场和出场。

![FreqUI - trade view](assets/freqUI-trade-pane-dark.png#only-dark)
![FreqUI - trade view](assets/freqUI-trade-pane-light.png#only-light)

### 图表配置器

FreqUI 图表可以通过策略中的 `plot_config` 配置对象（可通过"from strategy"按钮加载）或通过 UI 进行配置。
可以创建多个图表配置并随意切换——允许灵活地以不同视角查看您的图表。

图表配置可以通过交易视图右上角的"Plot Configurator"（齿轮图标）按钮访问。

![FreqUI - plot configuration](assets/freqUI-plot-configurator-dark.png#only-dark)
![FreqUI - plot configuration](assets/freqUI-plot-configurator-light.png#only-light)

### 设置

可以通过访问设置页面更改多项与 UI 相关的设置。

您可以更改的内容包括（等等）：

* UI 的时区
* 将未平仓交易显示为 favicon（浏览器标签页）的一部分
* K 线颜色（涨/跌 -> 红/绿）
* 启用/禁用应用内通知类型

![FreqUI - Settings view](assets/frequi-settings-dark.png#only-dark)
![FreqUI - Settings view](assets/frequi-settings-light.png#only-light)

## Web 服务器模式

当 freqtrade 以 [Web 服务器模式](utils.md#webserver-mode)启动时（使用 `freqtrade webserver` 启动 freqtrade），Web 服务器将以特殊模式启动，允许使用额外功能，例如：

* 下载数据
* 测试交易对列表
* [回测策略](#backtesting)
* ……更多功能待扩展

### 回测

当 freqtrade 以 [Web 服务器模式](utils.md#webserver-mode)启动时（使用 `freqtrade webserver` 启动 freqtrade），回测视图将变得可用。
此视图允许您回测策略并可视化结果。

您还可以加载和可视化之前的回测结果，以及将结果相互比较。

![FreqUI - Backtesting](assets/freqUI-backtesting-dark.png#only-dark)
![FreqUI - Backtesting](assets/freqUI-backtesting-light.png#only-light)


## CORS

整个这一部分仅在跨域情况下才需要（即您在 `localhost:8081`、`localhost:8082` 等上运行多个机器人 API），并且希望将它们合并到一个 FreqUI 实例中。

??? info "技术说明"
    所有基于 Web 的前端都受到 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)——跨源资源共享的约束。
    由于对 Freqtrade API 的大多数请求都必须经过身份验证，正确的 CORS 策略是避免安全问题的关键。
    此外，标准不允许对带有凭据的请求使用 `*` CORS 策略，因此此设置必须适当配置。

用户可以通过 `CORS_origins` 配置设置允许来自不同源 URL 的访问来访问机器人 API。
它由一个允许消费机器人 API 资源的 URL 列表组成。

假设您的应用程序部署在 `https://frequi.freqtrade.io/home/`——这意味着需要以下配置：

```jsonc
{
    //...
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": ["https://frequi.freqtrade.io"],
    //...
}
```

在以下（相当常见的）情况下，FreqUI 可通过 `http://localhost:8080/trade` 访问（这是您在导航到 freqUI 时在导航栏中看到的内容）。
![freqUI url](assets/frequi_url.png)

此情况的正确配置是 `http://localhost:8080`——URL 的主要部分（包括端口）。

```jsonc
{
    //...
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": ["http://localhost:8080"],
    //...
}
```

!!! Tip "尾部斜杠"
    `CORS_origins` 配置中不允许使用尾部斜杠（例如 `"http://localhots:8080/"`）。
    这样的配置不会生效，CORS 错误将会持续。

!!! Note
    我们强烈建议将 `jwt_secret_key` 设置为随机的且仅您自己知道的值，以避免对您的机器人进行未经授权的访问。
