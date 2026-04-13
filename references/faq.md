# FAQ

## Q: 部署这个 skill 时，用户一定要提供 SSH 登录信息吗？

不一定。

- 规划部署模式不需要登录信息
- 只有用户明确要求 AI 代执行部署时，才需要 SSH 地址、用户名和提权方式

默认应先让 AI 生成方案、模板、检查单和回滚方案。

## Q: 为什么优酷、爱奇艺、B 站提示“关闭 VPN”或“仅限大陆地区播放”？

常见原因：

- `fake-ip` 让域名返回 `198.18.x.x`
- 风控域名被 AdGuardHome 或代理规则误伤
- 应该直连的域名被送进代理

处理方向：

- 把视频平台和风控域名加入 `fake-ip-filter`
- 确保这些域名有对应的 `DIRECT` 规则
- 在 AdGuardHome 中放行关键风控域名，例如 `mmstat.com`

## Q: fake-ip 会带来哪些典型问题？

典型问题包括：

- 国内视频平台风控误判
- 腾讯游戏能进游戏但语音异常
- 微信图片加载慢或失败
- 某些服务网页能开，但登录、上传、下载或静态资源异常
- AI 平台子域名未进入 AI 专用组，落入默认代理组

## Q: 如何判断是不是 fake-ip 问题？

可以按下面方式判断：

1. 用 `dig @SIDE_ROUTER_IP DOMAIN +short` 看是否返回 `198.18.x.x`
2. 看这个域名是否在 `fake-ip-filter`
3. 看它是否还需要 `DIRECT`
4. 看应用是否依赖 UDP、真实 IP 或反代理校验

如果把域名加入 `fake-ip-filter` 后恢复，通常就是 fake-ip 兼容性问题。

## Q: 腾讯游戏或语音为什么容易出 fake-ip 问题？

因为这类业务常依赖：

- 真实 DNS 结果
- UDP 协议
- 语音或信令协商
- 区域和网络环境检测

它们不仅要 `DIRECT`，通常还要加入 `fake-ip-filter`。

## Q: AI 平台为什么网页能打开，但接口或静态资源异常？

常见原因：

- 只配置了 `claude.ai` 或 `openai.com`
- 漏掉了 `claudeusercontent.com`、`oaistatic.com`、`oaiusercontent.com` 等子域名
- 这些子域名未命中 AI 专用组，回落到默认组

解决方向：

- 为 AI 平台补齐子域名规则
- 确保命中专用 AI 故障迁移组

## Q: 重启旁路由后全网断网，但服务明明在运行，为什么？

高频原因：

- `iptables` 规则被持久化了
- 但 `ip rule` 和 `ip route` 没有持久化
- TPROXY 打了 mark，但没有对应策略路由处理，TCP 黑洞

解决方向：

- 使用 systemd 服务恢复 `ip rule add fwmark 1 table 100`
- 同时恢复 `ip route add local default dev lo table 100`

## Q: 没有网络时，应该先让 AI 排查还是先手动恢复？

先手动恢复一台测试设备联网。

最小动作：

1. 把测试设备网关改回主路由
2. 把测试设备 DNS 改回主路由或公共 DNS
3. 验证联网恢复
4. 再把诊断信息贴给 AI
