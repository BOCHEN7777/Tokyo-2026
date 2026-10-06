# Tokyo-2026 Firebase 数据访问设置

本项目把公开记账与私人行程协作分开：

- **公开记账**：所有访问者都可以查看并添加消费记录，不需要登录或共享房间。记录写入 Realtime Database 的 `/publicLedger/expenses`，只允许新增，不能修改或删除。
- **私人行程协作**：待办、愿望单、预约、行程参考资料等仍由 Firebase Authentication 白名单账号和共享房间保护。
- **地点照片**：只允许白名单账号通过 Cloud Storage 读取和写入。

公开账本中的项目名称、归属、类别、备注和金额会对任何网站访问者可见。不要记录银行卡、护照、验证码、订单号等私人信息。

## 1. 发布 Realtime Database 规则

Firebase 安全规则保存在 Firebase 服务器上。更新 GitHub 中的规则文件后，还需要到控制台发布一次：

1. 打开 [Firebase Console](https://console.firebase.google.com/) → **Tokyo-2026** → **Realtime Database → 规则**。
2. 将仓库根目录 [firebase-rules.json](firebase-rules.json) 的完整 JSON 粘贴并替换当前规则。
3. 如果继续使用私人行程协作，在 Realtime Database 数据中保留 `allowedUsers` 下两位用户的完整 UID，且值设为布尔值 `true`。
4. 发布规则。

`publicLedger/expenses` 规则允许匿名读取和逐笔新增；记录 ID 必须唯一，已有记录不能覆盖或删除。其余路径仍采用原有白名单规则，未列出的路径默认拒绝。不要把根级权限改成 `auth != null` 或公开整棵数据库。

规则发布后，重新载入攻略页面。页面会连接公开账本，并将当前浏览器中已有的消费记录逐笔并入；本机快照和原有记录都会保留。若消费记录分散在多台设备或不同浏览器，请分别在仍保留数据的原设备上打开新版页面，让本机数据完成合并后再清理旧设备数据。可在页面导出 CSV 留档。

## 2. 私人行程协作账号

此设置只适用于待办、预约、愿望单和行程资料等私人协作内容；公开记账不需要这些账号或共享房间。

1. 在 **Authentication → 登录方法** 启用“电子邮件地址/密码”。
2. 在 **Authentication → 用户** 添加两位同行人的账号，并记录各自 UID。
3. 在 Realtime Database 中设置：
   ```json
   {
     "allowedUsers": {
       "第一位用户的完整UID": true,
       "第二位用户的完整UID": true
     }
   }
   ```
4. 页面中登录获准账号，生成共享房间码并邀请同行人加入。

不要把密码写入网站或 GitHub。不要把 UID 白名单移除，除非确定不再使用私人行程同步。

## 3. Cloud Storage 访问规则

地点照片只开放给上述白名单账号。

1. 在 Firebase Console → **Storage → Rules**，粘贴仓库根目录 [storage.rules](storage.rules)。
2. 将规则中的 `request.auth.uid in ['', '']` 两个空字符串替换为两位用户的完整 UID。
3. 确认 Storage bucket 与网页配置相同，然后发布规则。

规则只允许白名单账号访问 `trips/{房间哈希}/itinerary/day1—day8/...jpg` 路径。票券二维码和行李照片仍只保存在各自设备。

## 4. 数据保存位置

| 数据 | 保存位置 | 访问方式 |
|---|---|---|
| 消费记录 | Realtime Database `/publicLedger/expenses`，并保留本机副本 | 所有人可查看；所有人可逐笔新增 |
| 预算与汇率 | 当前设备；启用私人共享房间时兼容同步 | 本机或白名单账号 |
| 待办、愿望单、预约、行程收藏链接 | Realtime Database `trips/{房间哈希}/state` | 白名单账号，同一房间 |
| 地点照片 | Firebase Cloud Storage | 白名单账号 |
| 票券二维码、行李照片 | 当前设备 | 当前设备 |
| 账号密码 | Firebase Authentication | 网站不保存密码 |

公开记录只追加，应用不提供编辑、删除或清空操作。若有金额录错，请另加一条说明记录；不要上传含个人资料的照片或文本。请定期导出 CSV 和完整 JSON 备份。

## 5. 排错

- **公开账本权限错误**：确认 `firebase-rules.json` 已在 Realtime Database → 规则中发布，然后刷新网页。
- **记录只保存在本机**：查看记账模块的公开同步状态；记录不会因同步失败而从本机删除。
- **私人旅行数据无法同步**：核对 `allowedUsers` 中的 UID、登录状态和共享房间码。
- **地点照片失败**：核对 Storage 已初始化、bucket 名称和 Storage Rules 中的两个 UID。
- **离线使用**：本机数据仍可查看和添加；重新联网后会重试公开账本同步。
