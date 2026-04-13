# 部署流程

## 1. 前置确认

- 确认旁路由有 LAN 入站网卡
- 确认默认路由指向主路由
- 确认主路由 DHCP 可改或客户端可手动指向旁路由
- 确认有回滚路径

常用检查：

```bash
ip a
ip route
```

## 2. Mihomo 主配置

使用 `proxy-providers` 架构，主配置应至少包含：

- `allow-lan: true`
- `external-controller`
- `mixed-port`
- `dns.listen: 0.0.0.0:1053`
- `enhanced-mode: fake-ip`
- `fake-ip-filter`
- `proxy-providers`
- `proxy-groups`
- `rules`

关键约束：

- AI 域名显式绑定到 `AI_US`
- 腾讯游戏、语音、微信、国内视频等需要同时考虑 `DIRECT` 和 `fake-ip-filter`

参考模板：

- `../assets/config/mihomo-config.template.yaml`

## 3. AdGuardHome

AdGuardHome 上游 DNS 指向 Mihomo：

- `127.0.0.1:1053`

对容易受广告规则误伤的域名做放行，例如：

- `@@||youku.com^$important`
- `@@||mmstat.com^$important`
- `@@||amdc.m.taobao.com^$important`
- `@@||qpic.cn^$important`
- `@@||gtimg.com^$important`

参考模板：

- `../assets/config/adguardhome-upstream.example.txt`

## 4. systemd 和订阅更新

推荐采用：

- `clash.service`
- `clash-provider-update.service`
- `clash-provider-update.timer`
- `clash-tproxy-route.service`

原则：

- 只允许 `systemctl` 管理 Mihomo
- 不要手工起第二个 Mihomo 进程
- 订阅更新只刷新 provider，不重写主配置

## 5. TProxy 与策略路由

部署 TProxy 时必须同时处理：

- `iptables -t mangle`
- `iptables -t nat` 的 DNS 劫持
- `ip rule add fwmark 1 table 100`
- `ip route add local default dev lo table 100`
- systemd 持久化策略路由

只保存 `iptables` 还不够；如果 `ip rule` / `ip route` 未持久化，重启后会出现 TCP 黑洞。

## 6. 上线验证

上线前至少验证：

```bash
/opt/clash/mihomo-bin -t -d /opt/clash -f /opt/clash/config.yaml
systemctl status clash --no-pager
systemctl status adguardhome --no-pager
curl -s http://127.0.0.1:9090/version
dig @SIDE_ROUTER_IP chatgpt.com +short
dig @SIDE_ROUTER_IP lol.qq.com +short
```

重点判定：

- AI 域名命中 AI 专用组
- 腾讯游戏和语音域名返回真实公网 IP，不是 `198.18.x.x`
- 本机控制口和局域网控制口都可访问
