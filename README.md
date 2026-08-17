# SNS Group Invitation QR Code Page

**期限切れのないSNSグループ招待QRコードを配信する静的ウェブサイト**
**A static website that always serves a valid SNS group invitation QR code**

<p align="center"><img src="screenshot.png" width="320" alt="Site screenshot"></p>
<p>
  <img src="https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white" alt="HTML5">
  <img src="https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white" alt="CSS3">
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black" alt="JavaScript">
  <img src="https://img.shields.io/badge/jsQR-lightgrey" alt="jsQR">
  <img src="https://img.shields.io/badge/GitHub%20Pages-222222?logo=githubpages&logoColor=white" alt="GitHub Pages">
  <img src="https://img.shields.io/badge/status-in%20operation-success" alt="In operation">
</p>

🔗 **Live site — [youngtheflash.github.io](https://youngtheflash.github.io/)**

**[中文版](#中文版) ・ [日本語](#日本語) ・ [English](#english)**

---

## 中文版

### 简介

这是一个**提供永久有效 SNS 群聊邀请二维码的静态网页**，主要面向即将赴日的新生，方便他们加入迎新答疑群。

作为宿舍管理员团队的一员，我发现了旧版邀请流程中存在的痛点。在会议上提出并达成共识后，我独立完成了该项目的方案设计、代码编写、部署上线及后续维护工作。

| | |
|---|---|
| **角色** | 独立开发（策划・设计・开发・运维） |
| **技术栈** | HTML / CSS / JavaScript (Canvas/jsQR) / GitHub Pages |
| **上线时间** | 2026年2月 〜 至今 |
| **用户** | 40余位即将入住的留学生，以及大学留学生科 |

### 解决的痛点

微信、LINE 等 SNS 群聊邀请二维码通常**只有约 1 周的有效期**。对于尚未赴日的新生来说，这种短期时效性给迎新工作带来了巨大的运营负担。

#### 引入前的情况

```
二维码过期失效
      ↓
宿舍管理员生成新的二维码
      ↓
通过邮件发送给大学留学生科
      ↓
留学生科向所有新生群发最新二维码
      ↓
（1周后，循环上述过程）
```

每次二维码过期，**宿舍方、留学生科、新生三方都会被迫进行重复沟通**。新生也经常会遇到“收到的二维码已经扫不出来”的尴尬局面。

#### 引入后的情况

```
留学生科只需发送一次“固定的网页链接”
      ↓
新生无论何时访问，都能看到当前有效的二维码
      ↓
宿舍管理员只需在后台替换图片文件（仅需 1 次 commit，约 30 秒）
```

| 项目 | 引入前 | 引入后 |
|------|--------|--------|
| 留学生科的工作 | 每次过期需配合群发 | **无需操作**（仅需首次通知链接） |
| 新生的体验 | 经常收到更新邮件 | **无需接收重复邮件** |
| 遭遇过期二维码 | 频繁发生 | **彻底杜绝** |
| 宿舍方的维护 | 繁琐的邮件沟通 | 仅需替换一张图片 |

### 设计亮点

#### 1. 链接固定，内容动态化

这是本项目的核心设计理念。通过**对外发布一个固定的网页链接，而在网页内部动态替换二维码图片**，从根本上解决了“邀请物料过期”的问题。

留学生科只需在迎新邮件中附上一次链接，后续的更新全部由前端页面自动处理。

#### 2. 彻底解决缓存问题 (Cache Busting)

上述设计隐藏着一个陷阱：由于每次替换的图片文件名保持不变（如 `qrcode_Wechat.JPG`），**浏览器很可能会直接加载本地缓存的旧图片**，导致新生依然看到过期的二维码。

由于 GitHub Pages 无法自定义 `Cache-Control` 响应头，我采用了在客户端动态生成唯一 URL 的方案：

```javascript
// 获取当前时间的毫秒数（一个永不重复的值）作为查询参数
// 强制浏览器每次都向服务器请求最新图片
var timestamp = new Date().getTime();
imgElement.src = src + '?t=' + timestamp;
```

每次请求的 URL 都不一样，从而完美避开了缓存机制。在无法修改服务器配置的受限环境下，巧妙地利用前端技术满足了业务需求。

#### 3. 智能二维码识别与高清裁切 (jsQR + Canvas)

微信或 LINE 生成的二维码图片往往带有大面积的白边，直接缩小显示会导致核心二维码区域过小、难以扫描。在新版本中，我引入了 **智能识别与裁切功能**：

- **引入 jsQR 库**：在前端直接解析图片数据，精准定位二维码的四个角坐标。
- **动态裁切**：根据坐标计算出紧凑的边界框，自动去除多余白边。
- **Retina 高清适配**：利用 `<canvas>` 以原始图片的真实物理像素大小进行裁切绘制，并配合 `image-rendering: pixelated;` 和 `ctx.imageSmoothingEnabled = false;` 关闭浏览器的抗锯齿模糊，确保即使在手机高清屏上，二维码边缘依然如刀切般锐利。

#### 4. 双平台无缝切换

页面支持微信群和 LINE 群的双平台切换。不仅二维码图片会随之改变，下方的提示文案也会根据平台特性动态更新（微信：进群后修改昵称；LINE：进群后私信或发送信息）。

#### 5. 极致的移动端体验与低成本运维

- **移动优先 UI**：使用 Flexbox 居中布局、动态宽度卡片，确保在任何尺寸的手机上都能完美显示。
- **零成本托管**：依托 GitHub Pages，无需服务器租用和维护费用。
- **单文件架构**：所有核心逻辑集中在一个 `index.html` 中，极易交接，后续管理员接手即用。

### 技术栈

| 分类 | 使用技术 |
|------|---------|
| 标记语言 | HTML5 |
| 样式表 | CSS3 (Flexbox, 响应式设计) |
| 脚本语言 | Vanilla JavaScript (Canvas API, Cache Busting) |
| 第三方库 | jsQR (用于前端二维码识别) |
| 托管平台 | GitHub Pages |
| 版本控制 | Git / GitHub |

### 文件结构

```
.
├── index.html           # 页面主体（包含所有 HTML/CSS/JS 及 Canvas 裁切逻辑）
├── qrcode_Wechat.JPG    # 微信群邀请二维码（定期替换）
├── qrcode_LINE.JPG      # LINE群邀请二维码（定期替换）
└── README.md
```

### 运维指南

更新二维码只需在本地替换对应的图片文件即可。

```bash
# 1. 将新的二维码图片保存为 qrcode_Wechat.JPG 或 qrcode_LINE.JPG 并覆盖旧文件
# 2. 提交并推送到 GitHub
git add qrcode_Wechat.JPG qrcode_LINE.JPG
git commit -m "更新二维码"
git push
```

推送后，GitHub Pages 会自动重新部署，约 1 分钟后生效。新生下次访问时，无需任何操作即可看到最新的二维码。

根据微信二维码的有效期，实际运维频率约为**每 7 天更新一次**。

### 总结与收获

虽然这是一个轻量级的项目，但我完整体验了**“用技术解决实际业务痛点”**的全生命周期：从发现问题、提出方案、沟通确认，到最终的开发上线和持续运维。

最大的收获在于：
- **直击本质**：不再试图优化“重新分发二维码的过程”，而是通过架构设计“消灭重新分发的需求”。
- **受限环境下的破局**：在无法控制后端的纯静态托管环境中，利用客户端技术（如时间戳防缓存、前端 Canvas 智能裁切）优雅地实现了复杂需求。

---

## 日本語

### 概要

来日を控えた中国からの入居者が、質疑応答用のSNSグループに参加するための
**招待QRコードを常に有効な状態で表示するウェブサイト**です。

寮の運営メンバーとして課題を発見し、ミーティングで提案・合意を得たうえで、
設計から実装・公開・運用までを個人で担当しました。

| | |
|---|---|
| **役割** | 個人開発（企画・設計・実装・運用） |
| **技術** | HTML / CSS / JavaScript (Canvas/jsQR) / GitHub Pages |
| **公開時期** | 2026年2月 〜 運用中 |
| **利用者** | 新入居予定の留学生 40名以上、および大学の留学課 |

### 解決した課題

SNSのグループ招待QRコードには**約1週間の有効期限**があります。
これが、来日前の新入生への案内フローにおいて大きな運用負荷を生んでいました。

#### 導入前

```
QRコードが失効
      ↓
寮のメンバーが新しいQRコードを生成
      ↓
留学課へメールで送付
      ↓
留学課が新入生全員へ一斉再送
      ↓
（1週間後、最初に戻る）
```

期限が切れるたびに、**寮側・留学課・新入生の三者すべてに再連絡の手間**が発生していました。
新入生側も「受け取ったQRコードがすでに使えない」という状態に頻繁に遭遇していました。

#### 導入後

```
留学課は「1本のURL」を一度だけ案内すればよい
      ↓
新入生はいつアクセスしても、常に有効なQRコードが表示される
      ↓
寮側は画像を差し替えるだけ（コミット1回・約30秒）
```

| 項目 | 導入前 | 導入後 |
|------|--------|--------|
| 留学課への依頼 | 失効のたびに毎回 | **不要**（初回の案内のみ） |
| 新入生への再送 | 失効のたびに一斉送信 | **不要** |
| 期限切れQRの受信 | 頻繁に発生 | **発生しない** |
| 運用作業 | 都度メール調整 | 画像を差し替えるのみ |

### 設計上の工夫

#### 1. URLは不変・中身だけ可変にする

本プロジェクトの中心となる考え方です。
**外部に配布するURLを固定し、その先の画像だけを差し替える**構成にしたことで、
「配布物が古くなる」という問題そのものを構造的に取り除きました。

留学課は最初の一度だけURLを案内すれば、以降の対応が一切不要になります。

#### 2. キャッシュ対策（タイムスタンプによる cache busting）

上記の設計には落とし穴があります。画像のファイル名が変わらないため、
**ブラウザが古い画像をキャッシュから表示してしまう**と、
せっかく差し替えても利用者には失効済みのQRコードが見えてしまいます。

GitHub Pages では独自の `Cache-Control` ヘッダを設定できないため、
クライアント側でアクセスのたびに一意なURLを生成する方式を採用しました。

```javascript
// 現在時刻のミリ秒（重複しない値）をクエリパラメータとして付与し、
// ブラウザに毎回サーバから画像を取得させる
var timestamp = new Date().getTime();
imgElement.src = src + '?t=' + timestamp;
```

これにより、リクエストURLが毎回異なるものとなり、キャッシュを確実に回避しています。
サーバ側の設定を変更できない制約の中で、クライアント側の実装で要件を満たした点が工夫箇所です。

#### 3. QRコードの自動認識と高画質クロッピング (jsQR + Canvas)

WeChatやLINEで生成されたQRコード画像には大きな余白が含まれており、そのまま縮小表示するとスキャンしづらくなる問題がありました。新バージョンでは以下の機能を実装しました：

- **jsQRライブラリの導入**：フロントエンドで画像データを解析し、QRコードの正確な座標を特定。
- **動的クロッピング**：座標から境界ボックスを計算し、不要な余白を自動的にトリミング。
- **Retinaディスプレイ対応**：`<canvas>`を用いて元画像の物理ピクセルサイズのまま描画し、`image-rendering: pixelated;` や `ctx.imageSmoothingEnabled = false;` を指定してブラウザのアンチエイリアスを無効化。これにより、スマートフォンでもエッジが鋭利な高画質QRコードを表示可能にしました。

#### 4. 複数プラットフォームへの対応とモバイルファーストUI

- **WeChat / LINEのシームレスな切り替え**：タブ一つで表示するQRコードを切り替え可能。プラットフォームに合わせて下部の案内テキスト（ニックネーム変更手順など）も動的に変更されます。
- **Flexbox による上下左右の中央配置** — 画面サイズを問わず視線の中心にQRコードが来る。
- **可変幅のカード**（`max-width: 320px` / `width: 85%`）— 小型端末でも見切れない。
- **GitHub Pages によるホスティング** — サーバ費用・保守作業ともにゼロ。

### 技術スタック

| 分類 | 使用技術 |
|------|---------|
| マークアップ | HTML5 |
| スタイリング | CSS3（Flexbox、レスポンシブ対応） |
| スクリプト | Vanilla JavaScript (Canvas API, cache busting) |
| ライブラリ | jsQR (フロントエンドでのQR認識) |
| ホスティング | GitHub Pages |
| バージョン管理 | Git / GitHub |

### ファイル構成

```
.
├── index.html           # ページ全体（HTML・CSS・JS・Canvas処理を内包）
├── qrcode_Wechat.JPG    # WeChat招待用QRコード（定期更新対象）
├── qrcode_LINE.JPG      # LINE招待用QRコード（定期更新対象）
└── README.md
```

### 運用方法

QRコードの更新は、画像の差し替えのみで完了します。

```bash
# 1. 新しいQR画像を qrcode_Wechat.JPG または qrcode_LINE.JPG として配置（上書き）
# 2. コミットしてプッシュ
git add qrcode_Wechat.JPG qrcode_LINE.JPG
git commit -m "renew QR code"
git push
```

プッシュ後、GitHub Pages が自動でデプロイし、約1分で反映されます。
有効期限に合わせて**約7日間隔で継続的に更新**しています。

### 成果と学び

技術的な規模は大きくありませんが、**「実際に困っている業務をITで解決する」**という一連の流れを完遂できました。

- **課題の本質を見極めること** — 「配り直し」を効率化するのではなく、「配り直しそのものを不要にする」設計への転換。
- **制約の中で要件を満たすこと** — バックエンドを持たない純粋な静的環境において、クライアント側の技術（時間戳キャッシュ回避、Canvasによる高画質トリミング）を駆使して複雑な要件を実現しました。

---

## English

### Overview

This is a **website that always displays a valid SNS group invitation QR code** for incoming
Chinese residents preparing to move to Japan, so they can join the Q&A group for new students.

As a member of the dormitory's operating team, I identified the problem, proposed the solution at
a residents' meeting, and — after gaining agreement — handled the design, implementation,
deployment and ongoing operation myself.

| | |
|---|---|
| **Role** | Solo project (planning, design, implementation, operation) |
| **Stack** | HTML / CSS / JavaScript (Canvas/jsQR) / GitHub Pages |
| **Launched** | February 2026 — currently in operation |
| **Users** | 40+ incoming international students, plus the university's International Student Office |

### The Problem It Solves

SNS group invitation QR codes **expire after roughly one week**. In the onboarding flow for
students still overseas, this created a significant recurring workload.

#### Before

```
QR code expires
      ↓
Dormitory member generates a new QR code
      ↓
Emails it to the International Student Office
      ↓
The office forwards it to every incoming student
      ↓
(One week later, repeat)
```

Every expiry forced **three parties — the dormitory, the office, and the students — to redo the
same communication**. Students frequently received a QR code that had already stopped working.

#### After

```
The office shares a single URL, once
      ↓
Students always see a valid QR code, whenever they visit
      ↓
The dormitory just swaps the images (one commit, ~30 seconds)
```

| | Before | After |
|---|--------|-------|
| Requests to the office | Every time it expired | **None** (one-time announcement) |
| Re-sending to students | Mass email on every expiry | **None** |
| Receiving expired QR codes | Happened frequently | **Eliminated** |
| Operational work | Email coordination each cycle | Replace the images locally |

### Design Decisions

#### 1. Keep the URL permanent, make only the content mutable

This is the central idea of the project. By **fixing the URL that gets distributed and swapping
only the image behind it**, the problem of "the thing you handed out goes stale" is removed
structurally rather than merely reduced.

The International Student Office announces the link once and never has to act again.

#### 2. Cache busting with a timestamp

That design has a pitfall. Because the image filename stays the same, **the browser may serve
a cached copy** — meaning users would still see an expired QR code even after the swap.

GitHub Pages does not allow custom `Cache-Control` headers, so I solved it on the client side by
generating a unique URL on every visit.

```javascript
// Append the current time in milliseconds (a value that never repeats)
// so the browser is forced to fetch the image from the server every time
var timestamp = new Date().getTime();
imgElement.src = src + '?t=' + timestamp;
```

Every request now targets a distinct URL, reliably bypassing the cache.

#### 3. Smart QR Code Recognition & High-Res Cropping (jsQR + Canvas)

QR codes generated by WeChat and LINE often include excessive white margins, making the actual code too small when scaled down. I introduced a smart cropping feature:
- **Client-side Recognition**: Utilises the `jsQR` library to parse the image data and locate the precise coordinates of the QR code.
- **Dynamic Cropping**: Calculates a bounding box to automatically strip away unnecessary white space.
- **Retina-ready Rendering**: Uses `<canvas>` to draw the cropped area at its original physical pixel size. By applying `image-rendering: pixelated;` and `ctx.imageSmoothingEnabled = false;`, it prevents the browser's default anti-aliasing, ensuring the QR code edges remain razor-sharp even on high-resolution smartphone screens.

#### 4. Dual Platform Support & Mobile-First UI

- **Seamless Switching**: Users can toggle between WeChat and LINE QR codes. The instructional text below updates dynamically based on the platform.
- **Flexbox centering on both axes** — the QR code sits at the visual centre on any screen size.
- **A fluid card** (`max-width: 320px` / `width: 85%`) — never clipped on small devices.

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 |
| Styling | CSS3 (Flexbox, responsive) |
| Scripting | Vanilla JavaScript (Canvas API, cache busting) |
| Libraries | jsQR (Front-end QR recognition) |
| Hosting | GitHub Pages |
| Version control | Git / GitHub |

### Project Structure

```
.
├── index.html           # The entire page (HTML, CSS, JS, Canvas logic)
├── qrcode_Wechat.JPG    # The WeChat QR code (rotated regularly)
├── qrcode_LINE.JPG      # The LINE QR code (rotated regularly)
└── README.md
```

### How It Is Operated

Updating the QR codes means replacing the image files.

```bash
# 1. Save new QR codes as qrcode_Wechat.JPG or qrcode_LINE.JPG
# 2. Commit and push
git add qrcode_Wechat.JPG qrcode_LINE.JPG
git commit -m "renew QR code"
git push
```

GitHub Pages deploys automatically and the change is live in about a minute. The images have been **rotated on a roughly 7-day cycle**, matching the expiry period.

### What I Learned

The technical scope is modest, but I was able to carry **"solving a real operational problem with technology"** all the way through myself.
- **Identifying the real problem**: The breakthrough was reframing the goal from "make redistributing the QR code more efficient" to "make redistribution unnecessary altogether."
- **Overcoming constraint limitations**: In a purely static environment with no backend control, I leveraged client-side technologies (timestamp cache-busting, Canvas-based smart cropping) to elegantly fulfil complex requirements.
