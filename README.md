# SpaceCloudNet — 主站

生产地址：**https://spacecloudnet.com/**（`www.spacecloudnet.com` 由 GitHub Pages 自动 301 到根域名）

## 这是什么

纯静态单页占位站：无构建步骤、无外部依赖（不引用任何 CDN、字体或统计脚本），GitHub Pages 直接托管 `main` 分支根目录。

## 文件

| 文件 | 作用 |
|---|---|
| `index.html` | 访问页面本体：站名、双语标语、联系入口；内联 CSS/JS |
| `404.html` | 自定义 404，保持与首页一致的视觉 |
| `CNAME` | 声明自定义域名 `spacecloudnet.com`（**不要删除**，GitHub Pages 靠它识别域名） |
| `favicon.svg` | 矢量图标，同时用作页面 logo |
| `robots.txt` | 允许搜索引擎收录 |
| `.nojekyll` | 跳过 Jekyll 处理，避免下划线开头的文件被忽略 |

## 改文案

所有会随语言切换的文字都写成一对属性，改属性值即可，两处同时改：

```html
<p class="lead" data-zh="中文文案" data-en="English copy">中文文案</p>
```

- 默认语言按浏览器语言判断，用户手动切换后记在 `localStorage.scn-lang`。
- 联系方式当前是 `moozaa75@163.com`，出现在 `mailto:` 链接和 `.addr` 文本里，改两处。
- 页脚年份由 JS 自动填充，不用手改。

## 发布

推送到 `main` 分支即自动重建，约 30–60 秒生效：

```powershell
git clone https://github.com/moozaa75/SpaceCloudNet.git
# 修改文件后
git add -A; git commit -m "Update copy"; git push
```

## DNS 记录（阿里云云解析 · 权威 NS `dns31/dns32.hichina.com`）

| 类型 | 主机记录 | 记录值 | TTL | 说明 |
|---|---|---|---|---|
| A | `@` | `185.199.108.153` | 600 | GitHub Pages 官方地址 |
| A | `@` | `185.199.109.153` | 600 | |
| A | `@` | `185.199.110.153` | 600 | |
| A | `@` | `185.199.111.153` | 600 | |
| AAAA | `@` | `2606:50c0:8000::153` ~ `8003::153` | 600 | 可选（IPv6） |
| CNAME | `www` | `moozaa75.github.io` | 600 | 已存在 |

不要在 `@` 上使用 CNAME：不符合 DNS 规范，且会与将来可能添加的 MX 记录冲突。

## HTTPS

证书由 GitHub Pages 自动签发（Let's Encrypt），签发后需在仓库 **Settings → Pages** 勾选 *Enforce HTTPS*。GitHub Pages 服务器在境外，因此**无需 ICP 备案**。

> 注意：即使域名解析正确，证书签发通常也要 1–15 分钟；在此期间访问会出现证书告警，属正常过程。

## 默认地址为什么打不开

绑定自定义域名后，`https://moozaa75.github.io/SpaceCloudNet/` 会返回 GitHub 的 **404 "Site not found"**——这是预期行为，不是配置错误。GitHub 只为**根域名 ↔ `www`** 之间自动建立跳转；项目站点的默认地址在设置了自定义域名后不再提供服务，访问一律走 `https://spacecloudnet.com/`。

（依据：[About custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages) —— 文档只承诺 apex 与 www 之间的自动跳转。）

## 建议的加固（可选）

在 **Settings → Pages → Custom domain** 点 *Verify* 会给出一个 `_github-pages-challenge-moozaa75` 的 TXT 记录值，把它加到云解析可防止域名被其他 GitHub 账号抢注。当前两个域名的 `protected_domain_state` 为 `null`（未验证、也未处于风险状态）。

