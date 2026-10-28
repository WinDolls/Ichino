# 致 AirFoundation/Ichino 作者的实测补充

> 我们基于你的仓库把协议在**官方 1.05.60 机台对应的 1.55 服务器**上全链路跑通，
> 并做成了独立的 Python 内核。以下是与你文档/代码有出入或值得补进 README 的点。
> 感谢开源，省了我们至少三天的逆向。

## ✅ 与你一致、我们实测再次确认的

| 项 | 说明 |
|---|---|
| 登录签名 | `SHA256(完整Keychip + 北京时间 + 盐).upper()`，盐一致，实测 errorID=0 |
| titlekey / 无 Content-Type | `titlekey: SDGB` ✓；带 Content-Type 确实回 57 |
| 42081 加密 | AES-256-CBC + zlib 与你 1.53 的实现一致，**1.55 服务器沿用** |
| isLogin 语义 | Preview 的 `isLogin=true` → 跳过 UserLoginApi ✓（returnCode=100 实测） |

## ⚠️ 我们踩到的、建议你补进 README 的点

1. **`Mai-Encoding` 实际值是 `1.55`**。我们反射游戏的 `Net.NetHttpClient` 实例字段
   `DefaultHttpHeaderEncryption` 实锤 = `"1.55"`（`Headers.Add("Mai-Encoding", "1.55")`）。
   你设置页 hint 写的 1.53 至少不是这台机台在用值，建议 hint 改成「1.55（或按服务器版本填）」。

2. **`ObfuscateParam = 8bF76dE9`**（我们是从游戏 `Net.Packet.Packet.ObfuscateParam`
   静态字段抠出来的）。你的 config 把它留给用户填，很多用户不知道填什么。

3. **URL 路径 = `/Maimai2Servlet/<md5>`，没有 `mE2s3Jhd`**。`mE2s3Jhd` 是
   GetGameSettingApi 响应里的 `movieServerUri`，与 API 路由无关——这是个历史迷惑项。

4. **42081 的 TLS 指纹门比你文档写的更狠**：aimedb 那边 WinHTTP 能过，但 42081 这边
   **Dart/BoringSSL 也是 200 空体**（我们实测过）。只有**逐字节复刻机台 ClientHello**
   （Unity 2018 的 85 个密码套件、无 ALPN、`c030 c02c c028…` 顺序）才放行。
   我们最后手搓了 TLS 1.2（ECDHE-RSA-AES128-GCM，模板从抓包提取），Python 两百多行搞定。
   如果你想让 Flutter 端直连官方 42081，这条基本是硬门槛。

5. **aimedb 的纯 HTTP:80 也能登录**（同 handler，签名相同）——Linux 服务器不用折腾
   WinHTTP/Schannel。443 则只发叶子证书，需要你文档里那段 GlobalSign 中间证书补链。

6. **`player/records` 用 GET**。我们一开始照 query/player 的习惯发 POST，405。

7. **`number` 头**：未登录 `0`，登录后观察到 `101`（从游戏 .NET HttpWebRequest 反射确认，具体语义我们也没完全吃透，但照抄有效）。

8. 游戏侧真实请求头只有这六条（除 Host/Cookie）：`User-Agent: <md5>#<uid>`、
   `Content-Type: application/json`、`charset: UTF-8`、`Mai-Encoding: 1.55`、
   `number`、`Content-Encoding: deflate`（body 本身已是 AES(zlib)，这个头只是摆设）。

## 🎁 回礼

我们把这套实现整理成了一份可开源的 Python 内核（登录/手搓 TLS/成绩拉取/推送水鱼），
如果你有兴趣可以参考或合并进文档：「开源交付_更新b50方案.md」（随附）。
你的 `data/id_mapping/music.json` 被我们直接用于 CN musicId → 歌名映射，很好用，谢谢。

—— 夕立 bot 开发组 🐋
