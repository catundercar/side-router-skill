# DNS 检查清单

- AdGuardHome 是否监听 `:53`
- Mihomo DNS 是否监听 `:1053`
- AdGuardHome 上游是否指向 `127.0.0.1:1053`
- 客户端 DNS 是否指向旁路由
- `fake-ip-filter` 是否覆盖高风险域名
- AdGuardHome 是否误拦关键风控域名
- `dig @SIDE_ROUTER_IP DOMAIN +short` 返回的是否是预期 IP
