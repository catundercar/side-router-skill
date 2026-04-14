# Side Router Skill

[English summary](#english-summary)

面向 Ubuntu 旁路由部署的开源 AI skill，核心场景是：

- 规划和审查 `AdGuardHome + Mihomo/Clash Meta + TProxy` 部署
- 生成部署步骤、配置模板、上线检查单和回滚方案
- 处理部署后的 DNS、网关、TProxy、`fake-ip`、AI 分流问题
- 在断网时先恢复网络，再继续让 AI 协助定位问题

## 软路由功能介绍

这个仓库不是单纯的代理规则集合，它对应的是一套“旁路由 / 软路由接管方案”。

在典型部署下，软路由会承担这些能力：

- 接管局域网设备的默认网关
  - 让客户端的出站流量先进入旁路由，再由旁路由统一决定直连或代理
- 接管局域网设备的 DNS
  - 由 `AdGuardHome -> Mihomo DNS` 统一做广告拦截、分流和域名解析策略
- 对 TCP 流量做透明代理
  - 通过 `TProxy + iptables + policy routing`，减少每台设备单独配代理的成本
- 把 AI 相关域名固定送入专用节点池
  - 例如把 `OpenAI`、`Claude`、`Cursor` 域名固定到美国或台湾节点组
- 把国内常用服务维持为直连
  - 包括视频、微信、腾讯游戏、语音链路等高兼容性场景
- 用 `proxy-providers` 管理订阅
  - 让节点更新与主配置解耦，避免覆盖手写的规则和分组

适合的网络形态：

- 家庭网络统一代理和分流
- 小型办公网络统一 DNS 与 AI 出口
- 需要“部分站点代理、部分站点直连”的局域网
- 不希望每台终端单独装代理客户端的环境

它的核心价值不是“翻墙”本身，而是把网关、DNS、分流策略、AI 出口和兼容性治理集中到一台 Ubuntu 旁路由上。

## 高风险提示

这是高风险网络改造 skill。

- 它可能影响默认网关、DNS、透明代理和整段局域网的可用性。
- 如果让 AI 直接执行部署，可能导致 SSH 断连、全网断网、DNS 全部失效。
- 默认应先使用“规划部署模式”，确认方案、检查项和回滚方案后，再进入代执行。
- 没有本地控制台或旁路由物理接入能力时，不要盲目执行高风险变更。

## 适用范围

- 主机系统：Ubuntu 旁路由
- 关键组件：`AdGuardHome`、`Mihomo/Clash Meta`、`iptables`、`systemd`
- 常见目标：
  - 让客户端网关和 DNS 指向旁路由
  - 让 AI 域名固定走台湾/美国节点池
  - 国内常用服务、腾讯游戏、语音链路默认直连
  - 使用 `proxy-providers` 管理订阅，避免覆盖主配置中的规则和分组

## 支持的 AI 工具

- `Codex`
  - 使用 [SKILL.md](./SKILL.md) 和 [agents/openai.yaml](./agents/openai.yaml)
- `Claude Code`
  - 使用 [.claude/agents/side-router-deploy.md](./.claude/agents/side-router-deploy.md)
- `Cursor`
  - 使用 [.cursor/rules/side-router-deploy.mdc](./.cursor/rules/side-router-deploy.mdc)
- 其他工具
  - 可直接加载 [AGENTS.md](./AGENTS.md) 或阅读本 README

## 安装方式

### 推荐：`npx` 一键安装

这个 skill 的首选安装方式是直接用 `npx` 通过 `skills` 安装器下发到对应 AI 工具。

安装到本机已支持的工具：

```bash
npx skills add catundercar/side-router-skill --skill side-router-skill -y
```

只安装到指定工具：

```bash
# Claude Code
npx skills add catundercar/side-router-skill --skill side-router-skill -a claude-code -y

# Codex
npx skills add catundercar/side-router-skill --skill side-router-skill -a codex -y

# Cursor
npx skills add catundercar/side-router-skill --skill side-router-skill -a cursor -y
```

同时安装到多个工具：

```bash
npx skills add catundercar/side-router-skill --skill side-router-skill \
  -a claude-code -a codex -a cursor -y
```

前提：

- 本机有 `node` / `npx`
- 已安装对应的 AI 工具
- 使用 `skills` 安装器支持的 agent 目录布局

### 手动安装

只有在不能使用 `npx skills add` 时，才建议手动复制。

- `Codex`
  - 读取 [SKILL.md](./SKILL.md) 和 [agents/openai.yaml](./agents/openai.yaml)
- `Claude Code`
  - 读取 [.claude/agents/side-router-deploy.md](./.claude/agents/side-router-deploy.md)
- `Cursor`
  - 读取 [.cursor/rules/side-router-deploy.mdc](./.cursor/rules/side-router-deploy.mdc)

## 两种使用模式

### 1. 规划部署模式

默认模式，不需要登录信息。

用户至少提供：

- 主路由 IP
- 计划中的旁路由 IP
- Ubuntu 版本
- LAN 网卡名
- 是否接管默认网关
- 是否接管 DNS
- 是否启用 `AdGuardHome` / `Mihomo` / `TProxy` / `fake-ip`
- 节点来源方式：订阅或本地静态配置
- AI 分流目标：美国、台湾或两者

skill 输出：

- 部署步骤
- 配置模板
- systemd 服务建议
- 上线前检查项
- 风险提示
- 回滚方案

### 2. 代执行部署模式

只有在用户明确授权并具备兜底手段时才使用。

额外需要提供：

- SSH 地址
- 登录用户名
- 提权方式：`sudo` 或 `root`
- 是否接受短时断网
- 是否有本地控制台或旁路由物理接入能力

不提供这些信息时，skill 只能输出部署方案，不能直接代执行。

## 用户输入模板

建议用户按下面格式向 skill 提供信息：

```text
目标：规划部署 / 审查现有部署 / 排查故障 / 断网恢复
主路由 IP：
旁路由 IP：
Ubuntu 版本：
LAN 网卡名：
客户端网关是否指向旁路由：
客户端 DNS 是否指向旁路由：
使用组件：AdGuardHome / Mihomo / TProxy / fake-ip
节点来源：订阅 / 本地配置
AI 分流目标：美国 / 台湾 / 美国+台湾
当前症状：
最近改动：
是否允许 AI 代执行：
是否有本地控制台兜底：
```

进入排障时，再追加：

- `ip a`
- `ip route`
- `ss -lntup`
- `systemctl status clash --no-pager`
- `systemctl status adguardhome --no-pager`
- `dig` / `nslookup` 测试结果
- 关键配置片段：`dns`、`fake-ip-filter`、`proxy-groups`、`rules`

## 推荐阅读顺序

1. [references/risk-policy.md](./references/risk-policy.md)
2. [references/deployment-inputs.md](./references/deployment-inputs.md)
3. [references/architecture.md](./references/architecture.md)
4. [references/deployment.md](./references/deployment.md)
5. [references/troubleshooting.md](./references/troubleshooting.md)
6. [references/offline-recovery.md](./references/offline-recovery.md)
7. [references/faq.md](./references/faq.md)

## fake-ip 特别说明

`fake-ip` 是这个 skill 的重点风险项之一。它能提升代理链路的一致性，但也容易引发：

- 国内视频站提示“关闭 VPN”
- 腾讯游戏或语音协商异常
- 微信图片、登录态、下载链路变慢或失败
- AI 平台网页能打开，但静态资源、上传下载或子域名异常

具体处理见 [references/faq.md](./references/faq.md)。

## 离线恢复原则

如果部署后没有网络，先恢复网络，再继续让 AI 诊断。

标准顺序：

1. 先找一台测试设备，不要一上来改全网
2. 临时把测试设备网关改回主路由
3. 临时把测试设备 DNS 改回主路由或公共 DNS
4. 验证基础联网恢复
5. 再把诊断信息贴给 AI

详细流程见 [references/offline-recovery.md](./references/offline-recovery.md)。

## 目录说明

- [SKILL.md](./SKILL.md)：Codex 入口
- [AGENTS.md](./AGENTS.md)：通用 agent 入口
- [.claude/agents/side-router-deploy.md](./.claude/agents/side-router-deploy.md)：Claude Code 入口
- [.cursor/rules/side-router-deploy.mdc](./.cursor/rules/side-router-deploy.mdc)：Cursor 规则入口
- [references/](./references/)：知识文档
- [assets/config/](./assets/config/)：模板和清单

## English Summary

This repository packages an open-source AI skill for Ubuntu side-router deployments using AdGuardHome, Mihomo/Clash Meta, and TProxy.

Primary use cases:

- deployment planning
- config generation and review
- pre-change validation and rollback planning
- post-deployment troubleshooting
- emergency recovery when DNS or gateway changes break connectivity

This is a high-risk networking skill. Default to planning mode first. Only use direct execution mode when the user explicitly provides SSH access, privilege model, outage tolerance, and an out-of-band recovery path.
