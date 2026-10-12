# SpaceCloudNet — 主站

生产地址：**https://spacecloudnet.com/**（`www.spacecloudnet.com` 由 GitHub Pages 自动 301 到根域名）

## 这是什么

纯静态单页：无构建步骤、无外部依赖（不引用任何 CDN、字体库或统计脚本），GitHub Pages 直接托管 `main` 分支根目录。页面主体是一张 Canvas 星链网络动效图。

## 文件

| 文件 | 作用 |
|---|---|
| `index.html` | 访问页面本体：Canvas 动画 + 站名 + 双语标语 + 联系入口；CSS/JS 全部内联 |
| `404.html` | 自定义 404，沿用同一深空视觉 |
| `CNAME` | 声明自定义域名 `spacecloudnet.com`（**不要删除**，GitHub Pages 靠它识别域名） |
| `favicon.svg` | 矢量图标，同时用作页面 logo |
| `robots.txt` | 允许搜索引擎收录 |
| `.nojekyll` | 跳过 Jekyll 处理，避免下划线开头的文件被忽略 |

## 改文案

页面上所有会随语言切换的文字都是 `data-zh` / `data-en` 成对存在，改属性值即可，两处同时改：

```html
<h1 data-zh="太空云网" data-en="SpaceCloudNet">太空云网</h1>
```

整块文案集中在 `index.html` 的 `<!-- ===== 文案区（只改这里） ===== -->` 与 `<!-- ===== 文案区结束 ===== -->` 之间。约定：**属性存在但留空**（如 `data-en=""`）表示该语言下隐藏这个元素，由 CSS `:empty{display:none}` 负责收起，不会留下空白。

- 默认语言按浏览器语言判断，用户手动切换后记在 `localStorage.scn-lang`。
- 联系方式当前是 `moozaa75@163.com`，出现在 `mailto:` 链接和 `.addr` 文本里，改两处。
- 页脚年份由 JS 自动填充，不用手改。

## 动效设计

Canvas 上同时跑四种运动，全部由 `requestAnimationFrame` 驱动、按帧间隔推进（不依赖固定帧率）：

| 元素 | 运动 |
|---|---|
| 星空 | 三层不同视差的星点缓慢漂移 + 明暗闪烁 |
| 节点 | 9 个（窄屏 5 个）节点缓慢漂移，相遇即连成链路 |
| 数据点 | 14 个（窄屏 6 个）光点沿链路流动，带 3 段渐隐拖尾 |
| 数据弧线 | 每 8.5 秒放出一条椭圆轨道上的流动弧段 |

**性能与可用性约束**（都已在代码中落实）：

- 光晕用**预渲染贴图**（64×64 径向渐变 canvas）绘制，避免每帧 `shadowBlur`——后者在移动端会直接掉帧。
- 设备像素比封顶 2（`Math.min(devicePixelRatio, 2)`），避免 3x 屏无谓的 4 倍像素开销。
- 系统开启「减少动态效果」时**只静态渲染一帧**，不启动动画循环；设置中途变化也会实时响应。
- 标签页切到后台（`document.hidden`）时暂停绘制，切回继续。
- 鼠标牵引只在 `(hover: hover) and (pointer: fine)` 设备上注册，触屏不监听指针事件以省电。
- 中央压暗（`.scrim`）保证动态背景下文字始终清晰；Canvas 标了 `aria-hidden="true"`，屏读器不会读到装饰内容。

**调手感**：`index.html` 内联脚本顶部的 `var T = {...}` 是唯一的动效参数块——节点数、数据点数量、连线距离、鼠标影响半径、弧线间隔、全局速度倍率（`speed`）都在那里。配色则在 CSS `:root` 的 `--fx-*` 变量里，JS 会读取这些变量，所以改配色不用碰脚本。

## 验证

```powershell
python tools/test_site_files.py     # 文件结构与静态断言 + 调用下面的 Node 测试
node   tools/test_animation.cjs     # 用假 DOM/Canvas 真跑 600 帧，断言动画行为
python tools/verify_sites.py        # 线上 DNS / HTTP / TLS 端到端验证
```

`test_animation.cjs` 在没有浏览器的环境里用 `vm` 造一个假 DOM，真实驱动 600 帧，断言：动画循环持续自调度、每帧清屏、绘制确实发生、弧线在 10 秒内出现、减少动效时不启动循环、隐藏标签页时停止绘制、DPR 封顶、中英文切换正确。**注意它验证的是行为，不是观感**——视觉效果需要在真实浏览器里看。


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

