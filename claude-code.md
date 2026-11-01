# Claude Code / Claude 分流规则

核对日期：2026-11-01。适用于 Mihomo / Clash.Meta，格式为 `classical` + `text`。

共 66 条规则：64 条域名规则和 2 条官方 API 入站 IP 网段。覆盖 Claude Code，以及 Claude 网页、桌面、Chrome 桥接、MCP 内容和 Artifacts 的已知基础设施。规则数不等于覆盖的主机数量：域名后缀还包含其子域名。

## 订阅引用

规则集地址：[claude-code.list](https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.list)。

将以下内容合并到现有配置。`🤖AI` 必须是配置里真实存在的策略组；不要再创建第二个 `rules` 或 `rule-providers` 顶层键。

```yaml
rules:
  # 如有公司专用规则，可保留在本条之前。
  # 本条放在广告拦截、Google AI、综合 AI 和通用分流之前。
  - "RULE-SET,ClaudeCode,🤖AI"

rule-providers:
  ClaudeCode:
    type: http
    behavior: classical
    format: text
    interval: 86400
    proxy: 🤖AI
    path: ./ruleset/ClaudeCode.list
    url: https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.list
```

客户端需要更新规则提供组，才会读取新的远程内容。部分客户端还有覆写规则，最终配置的顺序决定实际分流。该文件是规则集，不是包含代理节点的完整订阅。

## 来源与取舍

- [Claude Code 官方网络要求](https://code.claude.com/docs/en/network-config)：核心 API、账号认证、SSO、更新安装、包管理、插件、Chrome 桥接、可选遥测与 Artifacts 依赖。
- [Claude Desktop 官方网络要求](https://code.claude.com/docs/en/desktop#network-access-requirements)：补全 `claude.app`、动态 CDN、用户内容和 MCP 子域名。
- [Claude API 官方 IP 清单](https://platform.claude.com/docs/en/api/ip-addresses)：仅使用入站 `160.79.104.0/23` 与 `2607:6bc0::/48`。网页中的 IPv4 `/21` 在官方文档里属于出站地址，不作为本机访问 API 的目的地址清单；不引入 ASN 数据库或扩大到整个 IPv6 `/32`。
- [v2fly anthropic 分类](https://github.com/v2fly/domain-list-community/blob/master/data/anthropic)：补充开发、内容及关联域名；在文件中独立标注为社区分类，不将所有条目宣称为官方 CLI 必需端点。
- [Net.Coffee 清单](https://ip.net.coffee/claude/site.html)：其中列出的 21 个具体域名均包含在本规则的匹配范围内，另有 Sift 服务域名补充。该网页并非完整的官方网络清单。
- [现用 OverseasAI 规则](https://github.com/viewer12/OverseasAI.list/blob/main/rule/Clash/OverseasAI/OverseasAI.list)：保留 Claude 相关的 Statsig、网页验证等补充，不照搬其他 AI 产品的全部域名。
- [Datadog intake 文档](https://docs.datadoghq.com/real_user_monitoring/guide/proxy-rum-data/)、[日志 API](https://docs.datadoghq.com/api/latest/logs/)、[浏览器 SDK](https://datadoghq.dev/browser-sdk/)：覆盖 US5 和其他已公开商业区域的接收端点，避免使用任意包含 `datadog` 的域名关键词。
- [Sift 集成文档](https://developers.sift.com/docs)：将网页的 `sift` 关键词限定到 `sift.com` 与 `siftscience.com`；不能据此断言 Claude Code 当前一定使用 Sift。

没有加入 `DOMAIN-KEYWORD,datadog/sentry/sift`。例如 `sentry.example.com` 并不属于 Sentry，不能因为名字相似就强制采用 AI 策略。也没有全量代理 Cloudflare、Auth0、Google、AWS、Azure 等平台。

没有加入全局 NTP：NTP 同步时间，并不会返回当地时区。网页据此推导“必须代理 NTP 才能避免风控”的结论缺乏依据，参见 [RFC 5905](https://www.rfc-editor.org/rfc/rfc5905.html)。

## 共享主机的影响

域名分流无法按 HTTPS 路径识别“是不是 Claude 的请求”。`storage.googleapis.com`、`registry.npmjs.org`、列出的 GitHub 主机、`googlesource.com`、公共脚本和字体 CDN，以及 Statsig、Sentry、Intercom、Sift、Datadog 端点，被其他程序访问时也会采用同一策略。为兼容常见 Mihomo 客户端，Gerrit 使用 `googlesource.com` 后缀覆盖官方列出的 `*-review.googlesource.com`，同时也会覆盖其他该后缀的主机。

遥测和网页统计规则只决定流量走向，不会开启被用户关闭的遥测，也不意味着它们是核心 API 的必要依赖。若你希望拦截这些服务，可在本规则之前添加明确的拦截覆盖。

## 稳定使用

在 `🤖AI` 中直接选择一个可用的固定节点；如果仍选“节点选择”或“自动选择”，出口会跟随它们变化。规则组不能保证节点本身可用，也不能保证不同域名在服务端一定解析到相同出口 IP。

系统代理模式下，终端进程可能不读取 macOS 的系统代理设置。按 [Claude Code 官方说明](https://code.claude.com/docs/en/network-config#proxy-configuration) 设置 HTTP 代理环境变量，或确认客户端的 TUN 已接管该进程。使用当前模板的 HTTP 端口时，可在启动 Claude Code 前设置：

```sh
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
```

代理端口需与客户端实际监听一致。已有公司代理环境变量时应按公司网络要求处理，不能盲目覆盖；本规则不修改你的环境变量或系统设置。不要为解决分流问题关闭 TLS 证书校验。

自定义 `ANTHROPIC_BASE_URL`、Bedrock / Vertex / Foundry、第三方中转、远程 MCP、插件执行的工具和任意 WebFetch 目标没有有限的通用域名清单。出现这些用途时，需要补充实际服务域名；未知服务商不会自动成为本规则的一部分。

DNS 加密及是否绕过代理由客户端最终 DNS、TUN 和进程设置共同决定，单独增加域名规则不能证明“绝不泄漏”或“绝不封号”。原有公司专用规则仍优先，如有重叠需按你的业务要求调整顺序。

## 本次验证

- 使用本机 Mihomo v1.19.33，通过填入本地测试节点后的模板配置校验。
- 隔离运行 Mihomo，加载 27 个提供组；新规则集实际解析到 66 条。
- 122 个 Claude 域名 / IP 测试目标全部命中 `ClaudeCode → 🤖AI`，包括网页 21 个具体域名和官方清单的 34 个端点样例。
- 132 个相似域名及网段外地址未匹配新规则；61 条 Google AI 域名规则和 2 项公司分流的静态回归检查通过。Mihomo 实际连接记录也确认 Gemini 保持走 Google AI 组。
- 运行测试使用本地缓存的规则文件和本地代理接收端，不访问真实 Claude 服务。因此这些结果证明加载和分流匹配，不证明真实节点可用、账号可用或 DNS 不泄漏。订阅后台仍需填充 `proxies` 和 `__ALL_PROXIES__`。
