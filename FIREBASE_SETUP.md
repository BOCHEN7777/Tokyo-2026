# 东京攻略 Firebase 两人实时共享设置

本页适用于仓库 BOCHEN7777/Tokyo-2026 和 Firebase 项目 tokyo-2026-f2e23。网站把行程文字与链接存入 Realtime Database，把用户选择共享的景点、餐厅和门店照片存入 Cloud Storage。两者均只允许 Firebase Authentication 白名单内的两个 UID。

## 1. 建立两位邮箱账号

1. 打开 [Firebase Console](https://console.firebase.google.com/) → Tokyo-2026。
2. 进入 **Authentication → 登录方法**，启用 **电子邮件地址/密码**，关闭匿名登录。
3. 在 **Authentication → 用户** 添加两个人的邮箱账号。
4. 从用户表格复制两位用户各自的 **UID**。不要把密码写入网站或 GitHub。

## 2. 配置 Realtime Database 白名单与规则

1. 进入 **Realtime Database → 数据**，建立：

   ~~~json
   {
     "allowedUsers": {
       "第一位用户的完整UID": true,
       "第二位用户的完整UID": true
     }
   }
   ~~~

   true 必须是布尔值，不能加引号。

2. 进入 **Realtime Database → 规则**，用仓库根目录 [firebase-rules.json](firebase-rules.json) 全文替换并发布。
3. 不要使用根级 auth != null 规则；它会放行任何成功登录的账号。

这组数据库规则会验证 UID 是否位于 allowedUsers，并默认拒绝未列出的路径。

## 3. 配置 Cloud Storage 访问规则

1. Firebase Console 左侧进入 **Storage**。若尚未初始化，先按控制台步骤创建项目默认 bucket；bucket 必须与网站 Firebase 配置中的 storageBucket 相同。
2. 打开 **Storage → Rules**，粘贴仓库根目录 [storage.rules](storage.rules)。
3. 在规则中的 request.auth.uid in ['', ''] 这一行，把两个空字符串分别替换为第 1 步复制的完整 UID，例如：

   ~~~text
   request.auth.uid in ['第一位用户的完整UID', '第二位用户的完整UID']
   ~~~

4. 检查 UID 与 Realtime Database 的 allowedUsers 完全一致，再点击 **发布**。

规则文件默认使用两个空 UID，因此在填入你们的 UID 前会拒绝所有 Storage 读写。不要把条件改为泛化的 auth != null。

### Storage 文件访问边界

- 仅允许两位指定 UID 访问 trips/{房间哈希}/itinerary/day1—day8/... 下的 JPG 文件。
- 上传限制为 JPEG、单张不超过 5 MB；网页会先缩小照片并移除 EXIF 信息。
- 网页使用 Firebase Storage SDK 的授权读取，不生成公开下载 token。
- 小红书、官网、地图等参考链接与照片路径由 Realtime Database 同步。
- 票券二维码、护照、行李追踪照片仍只保存在各自设备，不放进共享相册。
- 删除地点照片卡时，网页会同时尝试删除 Storage 文件和同步记录。

## 4. 两台手机连接共享旅行

1. 两台手机都打开攻略，进入 **账号与实时同步**。
2. 第一台用获准邮箱登录，点击“复制 UID”，确认已在 allowedUsers；输入显示名“没头脑”或“不高兴”，生成房间码并连接。
3. 点击“复制同行邀请”，发给另一位同行人。
4. 第二台打开邀请链接，用另一位获准邮箱登录，填自己的显示名并连接。
5. 在任意一天展开“**小红书灵感 · 景点 / 门店照片**”：先保存一条参考链接，再上传一张不含个人资料的地点照片。
6. 确认另一台设备打开页面后看到链接和照片。照片打不开时，优先检查 Storage Rules 中的 UID、Storage bucket 和规则发布状态。

## 5. 数据同步与备份

| 内容 | 保存位置 | 同步方式 |
|---|---|---|
| 待办、预算、消费、愿望单、票券文字、行李文字、行程收藏链接、地点照片路径 | Realtime Database | 同一共享房间实时同步 |
| 用户主动上传的行程地点照片 | Firebase Cloud Storage | UID 白名单授权读取 |
| 票券二维码、行李照片、汇率缓存 | 当前设备 | 不上传 |
| 登录密码 | Firebase Authentication | 网站不保存密码 |

全站 JSON 备份会保留文字和照片路径等记录，但不会把照片文件本身打包。请不要在公开攻略正文中填写护照号码、银行卡、酒店确认号或票券二维码。

## 6. 页面访问与数据访问是两层权限

Firebase 规则保护实时数据和共享照片，不会保护静态网页本身。当前 Cloudflare 网站与 Public GitHub 仓库仍可公开访问；若整站也要只给两人使用，请按 [PRIVATE_ACCESS.md](PRIVATE_ACCESS.md) 将仓库改为 Private，并给正式域名配置 Cloudflare Access。

## 排错

- **PERMISSION_DENIED / 链接同步失败**：核对两位 UID 是否在 allowedUsers，并确认数据库规则已发布。
- **照片上传失败**：确认 Storage 已初始化、bucket 与网页配置一致、Storage Rules 中两位 UID 已填入并发布。
- **能登录但不能连接**：该邮箱可能尚未加入白名单；从网页复制该 UID 并添加到数据库。
- **另一台设备没有更新**：确认双方登录同一 Firebase 项目、使用同一房间码，并保持页面打开。
- **离线使用**：旅行文字仍可查看；云端同步和照片加载需要网络。

Firebase 访问规则需要在 Firebase Console 发布；将文件提交到 GitHub 本身不会替你发布 Firebase 规则。
