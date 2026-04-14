# 排障手册

## 最小诊断信息

优先收集：

```bash
ip a
ip route
ss -lntup
systemctl status clash --no-pager
systemctl status adguardhome --no-pager
curl -s http://127.0.0.1:9090/version
dig @SIDE_ROUTER_IP service.example.com +short
dig @SIDE_ROUTER_IP lol.qq.com +short
```

再补充关键配置片段：

- `dns`
- `fake-ip-filter`
- `proxy-providers`
- `proxy-groups`
- `rules`

## 高频故障模式

### 1. `127.0.0.1:9090` 不通，本机 IP 可通

常见原因：

- TProxy 规则误拦本机流量
- 对保留网段未做 `RETURN`

### 2. 指定服务网站打不开，但通用外站正常

常见原因：

- 指定业务域名没有单独绑定到专用策略组
- 该服务的静态资源、上传下载或 API 子域名漏配

### 3. 订阅更新后担心规则被覆盖

正确做法：

- 使用 `proxy-providers`
- 让更新只刷新 provider 文件
- 把 `proxy-groups` 和 `rules` 固定在主配置中

### 4. 重启后全网断网

常见原因：

- `iptables` 规则仍在
- 但 `ip rule` / `ip route` 没有持久化
- TPROXY fwmark 无对应路由表处理，TCP 被黑洞

### 5. 微信、ToDesk、语音类服务异常

优先检查：

- 是否启用了 `fake-ip`
- 相关域名是否加入 `fake-ip-filter`
- 是否需要 `DIRECT`
- 是否存在 UDP 协议不走 TProxy 但仍依赖真实 IP 的情况
