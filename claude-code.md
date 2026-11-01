# Claude Code 分流规则：网站清单

按用户在 2026-11-01 提供的 [Net.Coffee 清单](https://ip.net.coffee/claude/site.html) 整理，使用独立的 `🧩 Claude Code` 策略组。

共 25 条规则：原 `payload` 中的 22 条，加上 `GEOSITE,anthropic`、`DOMAIN-KEYWORD,sentry` 和 `GEOSITE,category-ntp`。文末与原清单重复的 `datadog`、`sift` 关键词只保留一次，匹配范围不变。

## 文件和订阅

- [claude-code.yaml](https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.yaml)：`payload` YAML 格式，新模板使用此文件。
- [claude-code.list](https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.list)：内容相同的文本格式，保留原链接供已有配置使用。

将以下内容合并到现有配置。组内的候选策略组须已经存在；节点占位符由订阅后台填充。已经存在 `🧩 Claude Code` 组时不必重复添加。

```yaml
# 合并到已有的 proxy-groups / rules / rule-providers，勿重复创建同名顶层键。
# 放在公司专用规则之后、广告拦截和其他宽泛规则之前。
proxy-groups:
  - name: 🧩 Claude Code
    type: select
    proxies:
      - 🚀 节点选择
      - ♻️ 自动选择
      - ☑️ 手动切换
      - __ALL_PROXIES__

rules:
  - "RULE-SET,ClaudeCode,🧩 Claude Code"

rule-providers:
  ClaudeCode:
    type: http
    behavior: classical
    format: yaml
    interval: 86400
    proxy: 🚀 节点选择
    path: ./ruleset/ClaudeCode.yaml
    url: https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.yaml
```

使用文本文件时，将提供组的 `format` 改为 `text`，`path` 改为 `./ruleset/ClaudeCode.list`，`url` 使用上面的 `.list` 链接。两份文件包含完全相同的 25 条规则。

规则文件通过 `🚀 节点选择` 下载；`RULE-SET` 把匹配到的连接交给 `🧩 Claude Code`。客户端需要刷新规则提供组，才会读到远程修改。

## 本版范围

保留所给清单的核心域名、指定 CDN、认证和内容主机、Sentry / Statsig / Datadog / Sift、Intercom、Fathom，以及下面的兜底：

```text
IP-CIDR,160.79.104.0/21,no-resolve
IP-CIDR6,2607:6bc0::/32,no-resolve
IP-ASN,399358,no-resolve
GEOSITE,anthropic
DOMAIN-KEYWORD,datadog
DOMAIN-KEYWORD,sentry
DOMAIN-KEYWORD,sift
GEOSITE,category-ntp
```

IP 段按用户贴出的 `/21` 和 `/32` 保留。本版以该清单及其兜底为范围，未额外纳入之前的 GitHub、npm、Google Storage、公共脚本和字体 CDN 规则。

关键词规则不区分请求来自哪个程序。其他程序访问包含 `datadog`、`sentry`、`sift` 的域名，或者命中这里的遥测、客服域名时，也会使用 Claude 组。GitHub、npm 和其他未命中本清单的连接继续按主配置的原有规则处理。

## 数据与客户端要求

适用于支持这些规则的 Mihomo / Clash.Meta。`classical` 提供组可使用域名、IP、ASN 和 GeoSite 规则，见 [Mihomo 规则集格式](https://wiki.metacubex.one/config/rule-providers/content/)。

客户端须能加载 `anthropic`、`category-ntp` 两个 GeoSite 分类及 ASN 数据。本机测试使用 Sparkle 缓存的 `geosite.dat` 和 `ASN.mmdb`；其他客户端若没有数据，可能需要联网下载。分类匹配范围取决于客户端实际使用的数据库。

`category-ntp` 是对时间服务域名的分类分流，不是对全部 UDP/123 流量的兜底。实际 UDP 转发取决于客户端和所选节点的支持；NTP 同步时间，这些分流规则不会修改电脑时区。

其他 AI 保留原有 `🤖AI` 分流，Google AI 保留原有独立组。第三方 API 中转、自定义 MCP 和任意工具访问的目标，仍须按实际域名另行处理。

## 本次验证

使用本机 Mihomo v1.19.33，当前模板的 20 个策略组、27 个提供组加载成功，Claude 提供组实际解析出全部 25 条规则。网站列出的 21 个具体域名，以及监控关键词、NTP 分类和指定 IP 范围样例均命中 Claude 组。

66 个正向匹配目标、42 个不匹配样例、8 个未纳入的公共服务主机，以及 ChatGPT / Gemini 的分流检查通过。61 项 Google AI 和 2 项公司规则的静态回归检查通过。

测试使用本地缓存规则、测试节点和固定测试解析，验证配置加载与匹配；未验证真实代理节点、Claude 账号、DNS 泄漏或 NTP UDP 连通性。
