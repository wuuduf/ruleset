# Claude Code / Claude 专用分流规则

核对日期：2026-11-01。适用于 Mihomo / Clash.Meta，使用 `classical` + `text`。

共 24 条规则：22 条 Claude / Anthropic 相关域名规则、2 条官方 API 入站 IP 网段。匹配流量使用独立的 `🧩 Claude Code` 组。

## 覆盖范围

- API、登录、OAuth、SSO、Claude 自有的安装更新地址、Chrome 桥接、MCP、用户内容和 Artifacts 自有主机。域名后缀会同时匹配其子域名。
- v2fly `anthropic` 分类里的关联域名，已在文件中单独标注为社区补充，不将它们全部视为官方 CLI 必需端点。
- `anthropic.auth0.com`、`anthropic.com.cdn.cloudflare.net`、`servd-anthropic-website.b-cdn.net`、`anthropic-com.ghost.io` 等明确指向 Claude 的第三方托管主机；匹配范围限于这些具体主机。
- 官方 API 入站地址：`160.79.104.0/23` 和 `2607:6bc0::/48`。IP 规则采用 `no-resolve`。

本次移除了 42 条共享服务规则，包括 GitHub、npm、Google Storage、Homebrew、Googlesource、公共脚本和字体 CDN，以及 Statsig、Sentry、Datadog、Sift、Intercom、Fathom、Cloudflare 通用验证地址。

这些公共请求继续由主配置里的其他规则处理，可能走节点选择、综合 AI、直连或拦截，取决于你原来的配置。移除专用规则并不等于将它们改为直连。Claude 访问这些公共服务时，也按同样的原有分流处理。

## 订阅引用

地址：[claude-code.list](https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.list)。

将以下内容合并到已有配置；策略组的候选组须在你的配置中存在，节点占位符由订阅后台填充。已存在 `🧩 Claude Code` 组时不必重复添加。

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
    format: text
    interval: 86400
    proxy: 🚀 节点选择
    path: ./ruleset/ClaudeCode.list
    url: https://raw.githubusercontent.com/wuuduf/ruleset/main/claude-code.list
```

规则文件本身通过 `🚀 节点选择` 下载。`proxy` 字段决定提供组下载文件的出口，流量分流由上面的 `RULE-SET` 条目决定。更新规则提供组后，客户端才会读取远程修改；这是规则集，不是包含节点的完整订阅。

## 来源

- [Claude Code 官方网络要求](https://code.claude.com/docs/en/network-config)：用于确认 Claude 核心服务；其中列出的公共服务按用户要求交给原有分流。
- [Claude Desktop 官方网络要求](https://code.claude.com/docs/en/desktop#network-access-requirements)：Claude 自有 CDN、用户内容及 MCP 子域名。
- [Claude API 官方 IP 清单](https://platform.claude.com/docs/en/api/ip-addresses)：采用入站网段，未将出站 IPv4 `/21` 用作访问 API 的目的地址清单。
- [v2fly anthropic 分类](https://github.com/v2fly/domain-list-community/blob/master/data/anthropic)：关联域名补充。
- [Net.Coffee 清单](https://ip.net.coffee/claude/site.html)：保留具体 Claude 主机，移出其中的通用遥测和客服服务。不再以覆盖该网页全部条目为目标。

## 使用

在 `🧩 Claude Code` 中选择可用的固定节点。终端需要通过客户端的 TUN 或 HTTP 代理进入 Mihomo；是否使用系统代理取决于进程自身。其他 AI 保留原有 `🤖AI` 分流，Google AI 保留原有独立组。

自定义 API 中转、Bedrock / Vertex / Foundry、远程 MCP 和任意 WebFetch 目标需要根据实际域名另行处理。加载与分流验证不能保证真实节点、账号和服务连通性。

## 本次验证

Mihomo v1.19.33 成功加载当前模板的 20 个策略组、27 个提供组；Claude 提供组解析出 24 条规则。64 个 Claude 域名 / IP 测试目标命中独立组，42 个移出的共享服务主机按原有规则匹配。ChatGPT、Gemini 的实际匹配检查，以及 61 项 Google AI、2 项公司规则的静态回归检查通过。

测试使用本地缓存规则、测试节点和固定的测试解析，验证配置加载与分流匹配，不代表真实节点、账号或服务连通性已验证。
