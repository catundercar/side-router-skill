# 架构概览

## 目标拓扑

核心目标：

- 局域网设备网关和 DNS 都可指向旁路由
- 旁路由统一完成 DNS 分流和 TCP 透明代理
- AI 域名固定走台湾/美国节点池并具备故障迁移
- 国内常用服务、腾讯游戏和语音链路默认直连

## 数据路径

DNS 链路：

`Client -> AdGuardHome:53 -> Mihomo DNS:1053`

透明代理链路：

`Client TCP -> TProxy:7893 -> Mihomo Rule/Group`

## 目录约定

- Mihomo：`/opt/clash`
- AdGuardHome：`/opt/adguardhome/AdGuardHome`
- UI：`/opt/clash/ui`

## 服务约定

- `clash.service`
- `adguardhome.service`
- `clash-provider-update.service`
- `clash-provider-update.timer`
- `clash-tproxy-route.service`

## 关键设计点

- 使用 `proxy-providers` 承载订阅，避免节点更新覆盖主配置中的 `proxy-groups` 和 `rules`
- AI 域名单独绑定 `AI_US` 或等价故障迁移组
- `fake-ip-filter`、`DIRECT` 规则和 AdGuardHome 放行规则必须一起维护
- TProxy 不只依赖 `iptables`，还依赖策略路由 `ip rule` / `ip route`
