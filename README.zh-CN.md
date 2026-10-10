# iPhone 使用蜂窝网络无法打开 HTTPS 网站？一次真实的 Nginx HTTP/2 排障案例

**语言：** [English](README.md) · [Русский](README.ru.md) · [Español](README.es.md) · [Português (Brasil)](README.pt-BR.md) · [简体中文](README.zh-CN.md)

**案例状态：** 在生产环境的 HTTPS 虚拟主机上启用 HTTP/2 后，同一部 iPhone 在关闭 VPN、使用蜂窝数据的情况下立即成功打开了网站。结果记录于 **2026 年 10 月 10 日**。本文发布时，观察时间尚不足 24 小时，**长期稳定性仍未得到验证**。

本案例涉及 [polishelp.ru](https://polishelp.ru/)。它是根据网站负责人实际观察整理的技术报告，**并非受控实验室研究**。

## 问题概述

排障期间，多部 iPhone 上的 Safari 在未使用 VPN、通过蜂窝网络访问时，会出现页面长时间加载或无法显示的情况；Wi-Fi 和 VPN 下可以访问。负责人还反馈，Android 通过移动网络访问正常。

故障表现并不完全一致：有些尝试在发送 HTTP 请求之前就中断；另一些尝试收到了 **HTTP 200** 和 HTML，浏览器却仍无法正确显示页面。

调查尝试了其他服务器和域名、简化的静态 HTML、JavaScript、gzip 压缩、Next.js 预取、TLS 配置检查以及抓包，但都没有找到稳定可靠的修复方法。故障发生时，生产环境的 Nginx 使用 **HTTP/1.1**。

随后，仅在一个独立的诊断虚拟主机上启用了 HTTP/2。负责人反馈，该地址在受影响 iPhone 的蜂窝网络环境下立即打开。获得这一线索后，又在**原生产域名及其 `www` HTTPS 虚拟主机**上启用了 HTTP/2。负责人确认，同一部 iPhone、同一移动网络环境下，原生产网址随即成功打开，同时保留了 HTTP/1.1 回退支持。

## 能证明什么，不能证明什么

在**同一生产域名和同一证书**上进行修改前后的观察，比不同域名之间的对比更有参考价值。这强烈提示：切换到 HTTP/2，或者旧 HTTP/1.1 传输路径与客户端网络环境的相互作用，与问题有关。

但这**并不能证明 Safari、Nginx、TLS、HTTP/1.1 或运营商存在某个特定缺陷**。手机端结果来自负责人的实际反馈，没有与测试同时进行的精确抓包；网络状况和浏览器状态也可能随时间改变。

此外，启用 HTTP/2 时需要对 Nginx 执行平滑重载，这也可能改变已有连接的状态。因此，仅凭一次成功打开，无法完全区分协议切换与重载的影响。底层根因仍未知。

这次生产改动没有更换域名、证书、DNS、TLS 安全策略、应用代码或网站内容。

## 详细技术资料

以下资料提供英文和俄文版本：

- [调查时间线（英文）](docs/timeline.md) · [俄文](docs/timeline.ru.md)
- [未奏效或结论不明确的尝试（英文）](docs/failed-attempts.md) · [俄文](docs/failed-attempts.ru.md)
- [诊断方法（英文）](docs/diagnostics.md) · [俄文](docs/diagnostics.ru.md)
- [Nginx HTTP/2 配置及回滚（英文）](docs/nginx-http2.md) · [俄文](docs/nginx-http2.ru.md)

## 实践中的注意事项

**HTTP/2 不是所有 iPhone 蜂窝网络故障的通用解决方案。** 在修改配置前，请确认 Nginx 版本及 HTTP/2 模块支持，备份现有配置，运行 `nginx -t`，仅修改目标 HTTPS 虚拟主机，并准备回滚方案。修改后应检查实际协商的 HTTP 版本，确认 HTTP/1.1 仍可用，并在真实受影响的设备与网络上多次验证。

Nginx 从 **1.25.1** 版本开始支持 `http2 on;` 指令，详情见[官方模块文档](https://nginx.org/en/docs/http/ngx_http_v2_module.html)。

## 隐私与许可

报告没有公开内部 IP、原始访问日志、管理系统私有路径、账号凭据或内部事故记录链接。请勿在公共 Issue 中上传包含用户信息的抓包文件。

本文档采用 [CC BY 4.0](LICENSE) 许可。
