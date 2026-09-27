# 将整站限制为两人访问

当前项目通过仓库中的 `wrangler.jsonc` 自动部署到 Cloudflare Workers，Workers 网址和 GitHub 源码仓库目前都是公开的。Firebase 登录只能保护待办、消费、预约等云端数据，不能阻止陌生人读取攻略正文或 Public 仓库中的 `index.html`。

若希望“整份攻略也只有你们两个账号能打开”，需要同时完成两层保护：把 GitHub 仓库改为 **Private**，并在 Cloudflare 为网站配置 **Access 邮箱白名单**。只做其中一项都不完整。

## 推荐方案：Private GitHub + Cloudflare Workers + Access

1. 在 GitHub 仓库 **Settings → General → Danger Zone → Change repository visibility**，把 `BOCHEN7777/Tokyo-2026` 改为 Private。先确认 Cloudflare GitHub App 对私有仓库仍有访问权限。
2. 在 Cloudflare 给 `tokyo-2026` Worker 绑定一个由 Cloudflare 托管的自定义域名；正式只分享这个域名。
3. 进入 **Zero Trust → Access → Applications → Add an application → Self-hosted**。
4. Application domain 填入刚才的自定义域名。
5. 建立 Allow 策略：Include 选择 **Emails**，只填写两个人的邮箱；登录方式可用一次性验证码或 Google。
6. 检查默认 `workers.dev` 地址是否仍能绕过 Access。若能，在 Worker 设置中关闭公开 `workers.dev` 路由，或让它同样受保护。
7. 用无痕窗口测试：未登录应被拦截，非白名单邮箱应被拒绝，两位获准邮箱可以进入。

## 如果继续保持公开部署

- 可以继续使用当前页面；攻略正文和 GitHub 源码是公开的，但 Firebase 私人数据在严格规则下只有白名单账号能读取。
- 不要在静态 HTML 正文中写护照号、手机号、酒店确认号、票券二维码或银行卡信息。
- 预约号等私人内容只放进登录后的“票券与预约夹”。

## 安全检查

- Firebase 匿名登录已关闭。
- Realtime Database 已发布仓库中的严格规则。
- `allowedUsers` 只有两个 UID。
- GitHub 源码中没有密码、身份证件、二维码或管理员密钥。
- Cloudflare Access 无痕测试通过后，再把私密资料迁入网页。
- 已安装的离线网页会在设备上缓存攻略正文；需要撤销某台设备的访问时，还要在该设备删除网站数据或移除主屏幕应用。
