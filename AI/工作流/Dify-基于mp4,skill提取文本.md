基于三个API接口，服务端需要启动 脚本start_dify.bat


```shell
 全部跑通了。端到端验证结果：

  ┌───────────────────────────────┬───────────────────────────────────────────────────────┐
  │             验证              │                         结果                          │
  ├───────────────────────────────┼───────────────────────────────────────────────────────┤
  │ 本机 /health                  │ 200 {"ok": true}                                      │
  ├───────────────────────────────┼───────────────────────────────────────────────────────┤
  │ 公网 /health（走 Cloudflare） │ 200 {"ok": true}                                      │
  ├───────────────────────────────┼───────────────────────────────────────────────────────┤
  │ 公网 POST 不带 key            │ 401（鉴权生效）                                       │
  ├───────────────────────────────┼───────────────────────────────────────────────────────┤
  │ 公网 POST 带 key              │ 404（鉴权通过，进入正常处理逻辑，仅因测试文件不存在） │
  └───────────────────────────────┴───────────────────────────────────────────────────────┘

  现在去 Dify 填这两处

  1. API Base URL（原 http://127.0.0.1:8765）改成：
  https://whether-times-clouds-article.trycloudflare.com  ==每次生成新的==
  2. 在工具设置里直接改；若该字段不可编辑，就把 openapi.yaml 的 servers.url 改成上面地址后重新导入。
  3. 鉴权选 Bearer Token，Token 填：
  ***REMOVED***
  4. （Dify 会按 Authorization: Bearer <token> 发送，正好匹配服务端校验；不要选 None。）

  然后点「测试」跑一次即可，SSRF 不会再拦截——现在访问的是公网 HTTPS 域名。

  ⚠️ 两个重要提醒

  5. 这个地址是临时的。 trycloudflare
  快速隧道每次重启都会换一个随机子域名。目前服务和隧道都在后台运行着、地址保持有效；一旦它们停了或重启，URL
  就变了，你得去 Dify 更新 base URL。

  以后要重新拉起来，执行这两条：
  # 终端 1
  cd "K:\AI_Test\AI_Skills\subtitle-screenshot-notes"
  python scripts/dify_server.py --host 0.0.0.0 --port 8765 --api-key ***REMOVED***

  # 终端 2
  /c/Users/Li/cloudflared/cloudflared.exe tunnel --url http://127.0.0.1:8765 --no-autoupdate
  （cloudflared 已装在 C:\Users\Li\cloudflared\cloudflared.exe，启动后从日志里抄新 URL。）

  6. 想要固定不变的地址：注册个免费 Cloudflare 账号，用 named tunnel + 你自己的域名（或 ngrok 的静态域名），就不用每次改
   Dify 了。

  需要我现在就用完整接口（拿真实视频/字幕）再从公网打一次 video_to_notes 确认全链路吗？还是你先去 Dify 填好配置试一下？

✻ Brewed for 10m 38s

● Background command "Download cloudflared exe and check version" failed with exit code 28

● 这是最开始那次被放弃的 GitHub 直连下载任务（b68c568）迟来的失败通知，exit 28 = 连接超时。那次下载早已弃用——后来已改用
  gh-proxy.com 镜像成功下载并验证了 cloudflared 2026.8.3。

  当前有效状态不受影响：
  - 工具服务 + 隧道都在后台正常运行
  - 公网地址 https://whether-times-clouds-article.trycloudflare.com 已验证可通（health 200、鉴权 401/通过 均正常）

  无需处理这条通知。你可以去 Dify 填配置试一下；需要我再用完整接口从公网打一次 video_to_notes 验证全链路也可以说。

● Background command "Download cloudflared via ghproxy.net mirror" failed with exit code 18

● 同样是已被绕过的旧下载任务的通知：b9d59f9 是 ghproxy.net 镜像那次（exit 18 = 传输中断/部分文件）。那次失败后我已改用
  gh-proxy.com 并断点续传成功，最终下载并验证了完整的 cloudflared 2026.8.3（55MB）。

  当前有效状态不受影响，隧道与工具服务仍在运行。无需处理这条通知。
```