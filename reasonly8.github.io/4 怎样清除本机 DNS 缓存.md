本地计算机会将最近查询过的 DNS 记录进行缓存，如果你修改了域名的 DNS 解析规则，那你的本机会因为缓存的原因无法正确访问最新的记录，这时就需要清除本机缓存。

```sh
# Windows - powershell
ipconfig /flushdns

# MacOS
sudo killall -HUP mDNSResponder

# Linux
sudo systemctl restart systemd-resolved
```

注意：本机 DNS 缓存的存活时间也受 TTL 控制。
