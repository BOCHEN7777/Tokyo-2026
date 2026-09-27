# 将整站限制为两人访问

当前 GitHub Pages 地址是公开静态网页。Firebase 登录只能保护待办、消费、预约等云端数据，不能阻止陌生人读取公开攻略正文。

若希望“整份攻略也只有你们两个账号能打开”，推荐把同一仓库部署到 **Cloudflare Pages**，再用 **Cloudflare Access** 做邮箱白名单。不要只给 GitHub 仓库改成 Private：GitHub Pages 的可用性取决于套餐，而且这不是稳定的登录网关方案。

## 推荐方案：Cloudflare Pages + Access

1. 在 Cloudflare Dashboard 进入 **Workers & Pages → Create → Pages → Connect to Git**。
2. 选择 `BOCHEN7777/Tokyo-2026` 仓库；这是纯静态站点，无需构建命令，输出目录使用仓库根目录。
3. 等待首次部署，记下 `*.pages.dev` 地址。
4. 进入 **Zero Trust → Access → Applications → Add an application → Self-hosted**。
5. Application domain 填入 Pages 自定义域名。为避免平台地址绕过 Access，正式使用时应绑定自己的域名，并只分享该域名。
6. 建立 Allow 策略：Include 选择 **Emails**，只填写两个人的邮箱；登录方式可用一次性验证码或 Google。
7. 用无痕窗口测试：未获准邮箱应被拦截，两个获准邮箱能进入。

## 如果继续使用 GitHub Pages

- 可以继续使用当前页面；攻略正文是公开的，但 Firebase 私人数据在严格规则下只有白名单账号能读取。
- 不要在静态 HTML 正文中写护照号、手机号、酒店确认号、票券二维码或银行卡信息。
- 预约号等私人内容只放进登录后的“票券与预约夹”。

## 安全检查

- Firebase 匿名登录已关闭。
- Realtime Database 已发布仓库中的严格规则。
- `allowedUsers` 只有两个 UID。
- GitHub 源码中没有密码、身份证件、二维码或管理员密钥。
- Cloudflare Access 无痕测试通过后，再把私密资料迁入网页。
