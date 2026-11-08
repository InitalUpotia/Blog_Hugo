# 博客写作与维护手册

> 本文件放在 `docs/` 下，不参与 Hugo 站点构建，仅作为个人备忘。
> 最后核对时间：2026-11-08。

---

## 0. 一句话概览

- 框架：Hugo 0.167 extended + PaperMod（Git submodule）
- 部署：GitHub 仓库推送到 `main` 后，Vercel 自动执行 `hugo --gc --minify`
- 写作：`content/posts/` 下建 Page Bundle，图片和 `index.md` 放同一目录
- 发布：`draft: false` → commit → push → 等 Vercel 构建完成
- `public/`、`resources/_gen/`、`.hugo_build.lock` 已在 `.gitignore`，不要提交

---

## 1. 本地环境与常用命令

### 1.1 环境

- Hugo extended ≥ 0.146（当前 0.167.0）
- Git
- Go（仅 Hugo Modules 用，当前用于 `hugomods/shortcodes`）

### 1.2 常用命令

```bash
# 本地预览（包含草稿）
hugo server -D

# 只构建，检查错误和重复路径
hugo --gc --minify --printPathWarnings

# 新建文章（Page Bundle）
hugo new content posts/my-post/index.md

# 发布
git add -A
git commit -m "post: 文章标题"
git push
```

> `hugo server` 默认不显示草稿，调试草稿要加 `-D`。
> 构建产物在 `public/`，已被 Git 忽略；本地预览直接看 `hugo server`，不要用 `public/` 当预览源。

---

## 2. 新建一篇文章

### 2.1 推荐目录结构

```text
content/posts/
└── my-post/
    ├── index.md
    ├── cover.webp
    ├── 1.webp
    └── 2.webp
```

这种叫 Page Bundle，文章里的图片和封面直接写文件名即可，不会和别的文章串图。

也可以用单文件 `content/posts/my-post.md`，但图片只能放 `static/`，路径更容易写错，非必要不推荐。

### 2.2 Front Matter 模板

运行 `hugo new content posts/my-post/index.md` 会按 `archetypes/default.md` 生成：

```yaml
---
date: '2026-11-08T20:00:00+08:00'
draft: true
title: '文章标题'
author: 'iNITAL'
tags: []
# columns: ['iNITAL 的垃圾站']
# cover:
#   image: "cover.webp"
#   alt: "封面说明"
#   caption: "显示在封面下方的图注"
---
```

常用字段：

| 字段 | 说明 |
|---|---|
| `title` | 文章标题 |
| `date` | 发布时间，保留时区 `+08:00` |
| `draft` | `true` 不发布，`false` 发布 |
| `author` | 作者 |
| `tags` | 标签数组，会生成 `/tags/` 页面 |
| `columns` | 专栏名数组，值必须和 `content/columns/` 下的目录名一致 |
| `description` | 搜索摘要/分享摘要，建议每篇都写 |
| `math` | 需要公式时写 `math: true` |
| `cover` | 封面图，`image` 写 bundle 内的文件名或 `/static` 路径 |
| `lastmod` | 需要覆盖自动"最后更新"时间时才写 |

> 专栏的 front matter key 是 **`columns`**（复数），不是 `column`、`category`。
> 对应配置在 `hugo.yaml` 的 `taxonomies: category: columns`。

### 2.3 发布前把 `draft` 改成 `false`

```yaml
draft: false
```

`hugo` 正式构建不会输出 `draft: true` 的文章。

---

## 3. 正文写法

### 3.1 图片

Page Bundle 内直接写文件名：

```markdown
![](1.webp "~ 这是图注，会显示在图片下方 ~")
```

- 带 `"标题"` 会渲染成 `<figure>` + `<figcaption>`；
- 不带标题就是普通 `<img>`；
- 渲染钩子已经自动加 `loading="lazy"` 和 `decoding="async"`；
- **不会**做 Hugo 压缩/缩放，纵横比按原图显示；
- 建议上传前自己压成 WebP（Squoosh、ImageOptim 等），单张控制在 300KB 以内。

如果要放大图看原图，目前没有开启点击放大；需要的话可以在主题层面加 lightbox，不属于写作流程。

### 3.2 封面

```yaml
cover:
  image: "cover.webp"
  alt: "封面替代文字"
  caption: "封面图注"
```

注意：封面图仍然会被 PaperMod 处理（生成响应式 srcset）。如果封面比例也被压坏，在 `hugo.yaml` 加：

```yaml
params:
  cover:
    responsiveImages: false
```

正文图片不受这个开关影响。

### 3.3 数学公式

在 front matter 加：

```yaml
math: true
```

支持：

```markdown
行内：$E = mc^2$

块级：
$$
\int_0^1 x^2 \, dx
$$
```

不写 `math: true` 的页面不会加载 KaTeX，这是有意的性能优化。

### 3.4 网易云音乐 / Bilibili

```markdown
{{< netease-cloud-music id=22805132 type=song height=80 width=500 title="Masquerade" >}}

{{< bilibili "BV1uhYueJErx" >}}
```

这两个短代码来自 `hugomods/shortcodes` 模块；如果以后删掉模块，需要把短代码模板复制到 `layouts/shortcodes/`。

### 3.5 Markdown 扩展语法

当前启用了：

```markdown
==高亮==
++插入++
^上标^
~下标~
~~删除~~
```

原始 HTML 默认**不会**渲染（Goldmark `unsafe` 未开启）。确实需要 `<u>`、`<details>` 这类标签时，在 `hugo.yaml` 的 `markup.goldmark` 下加：

```yaml
renderer:
  unsafe: true
```

只在自己的内容里用，不要渲染外部来源的 HTML。

### 3.6 表格、引用、代码块

正常 Markdown 即可。代码块会自动带复制按钮（`ShowCodeCopyButtons: true`）。

---

## 4. 专栏（columns）

### 4.1 新建专栏

1. 新建目录和介绍页：

```text
content/columns/我的新专栏/_index.md
```

```yaml
---
title: '我的新专栏'
---
这里写专栏介绍，会显示在专栏页顶部。
```

2. 文章 front matter 引用：

```yaml
columns: ['我的新专栏']
```

3. 至少给该专栏下的一篇文章加封面，专栏卡片会取"最新一篇有封面的文章"的图作为缩略图。

### 4.2 注意

- 专栏目录名一旦上线，尽量不要再改，改了 URL 会变；
- 旧的专栏 URL 如果要保留，需要在 `vercel.json` 里加 `redirects`；
- 专栏名大小写由模板里的 `.Page.Data.Term` 保留，不需要额外处理。

---

## 5. 站点定制说明

### 5.1 背景图

位置：`layouts/partials/extend_head.html`

- 调 `https://t.alcy.cc/ycy?json` 拿一张固定图片 URL；
- 存进 `sessionStorage`，同一标签页内翻文章不变；
- 关闭标签页/浏览器后重新随机；
- 接口失败时退回直链，这时每页可能变化。

换图源就改这里的 URL。想按天变化而不是按会话变化，把 `sessionStorage` 换成 `localStorage` 并自己加日期 key。

### 5.2 背景蒙版与毛玻璃

位置：`assets/css/extended/custom.css`

```css
--bg-mask: rgba(255, 255, 255, 0.4);  /* 浅色背景蒙版 */
--bg-mask: rgba(0, 0, 0, 0.6);        /* 深色 */
--glass-bg: rgba(255, 255, 255, 0.4); /* 卡片/文章玻璃底色 */
--glass-blur: 4px;                    /* 模糊半径，移动端 2px */
```

- 调透明度就改这几个 alpha；
- 深色模式选择器必须是 `[data-theme="dark"]`，不要写 `.dark`（PaperMod 没有 `.dark` 类）；
- 代码块保持不透明，保证可读性。

### 5.3 评论（Giscus）

位置：`layouts/partials/comments.html`

- 仓库、分类等 ID 直接写死在这里；
- 主题用 `noborder_light` / `noborder_dark`，随站点深浅色切换；
- 文章底部评论是立即加载的，**不要**加 `data-loading="lazy"`，Giscus 的 iframe 初始高度为 0，懒加载会永远不触发；
- 评论区外框圆角在 `custom.css` 的 `.comments` / `iframe.giscus-frame` 里。

### 5.4 导航栏与下拉菜单

位置：`layouts/partials/header.html`

- 这是整份主题 header 的复制版，为了支持"另我空间"下拉菜单；
- 更新主题时**不要**直接用主题文件覆盖它，否则下拉菜单会消失；
- 菜单项在 `hugo.yaml` 的 `menu.main` 里改；
- 下拉触发目前是 `<a href="javascript:void(0)">`，只有鼠标悬停；需要键盘/触屏更友好时改成 `<button>` 并给 CSS 加 `:focus-within`。

### 5.5 文章详情页

位置：`layouts/_default/single.html`

- 复制了主题的 `single.html`，只为了在 meta 里加"阅读量"；
- 同样是主题更新时需要手动同步的文件。

### 5.6 最后更新时间

- `hugo.yaml` 的 `frontmatter.lastmod` 优先读 Git 提交时间；
- 本地正常；Vercel 是浅克隆，如果线上显示的是构建当天，说明 Git 历史没拿到；
- 解决：在文章 front matter 手写 `lastmod: '2026-11-08'`，或把配置改成 `:fileModTime`。

### 5.7 搜索

- `content/search.md` 提供 `/search/` 页面；
- `hugo.yaml` 的 `outputs.home` 包含 JSON，主题的 fastsearch 读取 `/index.json`；
- 不要删掉 `outputs` 里的 `JSON`。

### 5.8 访问统计与运行天数

- 位置：`extend_head.html`（Vercount 脚本）、`extend_footer.html`（访客数、运行天数）；
- 运行天数从 `new Date("2026-10-19T00:08:00")` 开始算，改建站日期就改这里。

---

## 6. 发布流程 Checklist

发布一篇新文章前逐项确认：

- [ ] `draft: false`
- [ ] `title`、`date` 正确
- [ ] `tags` 已填
- [ ] 需要进专栏的话，`columns` 的名字和 `content/columns/` 目录一致
- [ ] `description` 已写（影响搜索结果和分享预览）
- [ ] 封面 `cover.image` 文件存在
- [ ] 正文图片都在 bundle 里，路径没有拼错
- [ ] `hugo server -D` 本地预览过
- [ ] `hugo --gc --minify --printPathWarnings` 没有报错
- [ ] `git status` 里没有不想提交的文件
- [ ] commit 信息写清楚，push 后确认 Vercel 构建成功
- [ ] 打开线上页面，硬刷新确认图片和评论正常

```bash
hugo --gc --minify --printPathWarnings
git add -A
git commit -m "post: 文章标题"
git push
```

---

## 7. 常见问题

### 7.1 评论不加载，只显示浏览器占位/错误页

- 检查有没有误加 `data-loading="lazy"`，有就删掉；
- F12 → Network 搜 `giscus`，看 `client.js` 和 iframe 是否 200；
- F12 → Console 看 CORS / CSP 报错；
- 无痕窗口测试，排除 uBlock、AdGuard、Firefox 严格跟踪保护拦截 `giscus.app`。

### 7.2 背景图每翻一页就变

- 正常情况下同一标签页应该不变；
- 如果接口请求失败，会走 `.catch` 里的直链兜底，直链每次请求都会随机；
- 检查 `sessionStorage` 是否被浏览器策略禁用（隐私模式、第三方 Cookie 限制）。

### 7.3 图片比例不对

- 正文图片现在不做任何处理，按原图显示；
- 封面仍然会被主题处理，必要时设置 `params.cover.responsiveImages: false`；
- 手机拍的照片注意 EXIF 方向，上传前最好先导出成正常方向。

### 7.4 构建日志出现 "Raw HTML omitted"

- 正文里用了原生 HTML；
- 改用 Markdown 扩展语法，或开启 `markup.goldmark.renderer.unsafe: true`。

### 7.5 线上语言、日期变成英文

- 检查 `hugo.yaml`：

```yaml
defaultContentLanguage: "zh"
languageCode: "zh-CN"
```

- 修改后要 commit + push，Vercel 重新构建才生效。

### 7.6 Vercel 构建失败

优先看构建日志：

- Go 版本问题：`go.mod` 的 `go` 指令高于 Vercel 镜像版本；
- 主题 submodule 没拉下来；
- 模板语法错误（本地 `hugo` 会先报）。

临时规避 Go Modules：把 `netease-cloud-music`、`bilibili` 两个短代码复制到 `layouts/shortcodes/`，删掉 `hugo.yaml` 的 `module.imports` 和 `go.mod`、`go.sum`。

### 7.7 修改 CSS 后没变化

- Hugo 会把 CSS 指纹化，浏览器可能缓存旧文件；
- 硬刷新（Ctrl+F5）或看 Network 里 CSS 文件名 hash 是否变了；
- 改了 `hugo.yaml` 或模板后需要重新构建，`hugo server` 一般会自动热更新。

---

## 8. 自定义文件地图

| 文件 | 作用 | 什么时候改 |
|---|---|---|
| `hugo.yaml` | 站点配置、菜单、参数、Markdown 扩展 | 改菜单、开关功能、语言、分页 |
| `vercel.json` | 构建命令、缓存头、重定向 | 改构建参数、旧链接跳转 |
| `.gitignore` | 忽略构建产物 | 一般不动 |
| `assets/css/extended/custom.css` | 全部自定义样式 | 调毛玻璃、字体、专栏卡片、评论区圆角 |
| `layouts/partials/header.html` | 导航 + 下拉菜单 | 改菜单结构、可访问性 |
| `layouts/_default/single.html` | 文章详情模板 | 加阅读量等 meta 信息 |
| `layouts/partials/post_meta.html` | 日期、最后更新 | 改日期显示规则 |
| `layouts/_default/_markup/render-image.html` | 图片渲染 | 加懒加载、图注格式 |
| `layouts/partials/comments.html` | Giscus 评论 | 换评论仓库、主题 |
| `layouts/partials/extend_head.html` | KaTeX、Vercount、背景图 | 换背景 API、统计脚本 |
| `layouts/partials/extend_footer.html` | 页脚、运行天数 | 改建站日期、页脚文案 |
| `layouts/columns/terms.html` | 专栏卡片 | 改专栏卡片布局 |
| `archetypes/default.md` | 新文章模板 | 改默认 front matter |
| `content/columns/` | 专栏介绍页 | 新建/修改专栏 |
| `static/` | 静态图片、logo | 放全局图片 |
| `themes/PaperMod/` | 主题 submodule | 不直接改，更新用 `git submodule` |

---

## 9. 不要做的事

1. **不要提交 `public/`**：它是构建产物，而且可能带着 localhost 的 canonical。
2. **不要直接改 `themes/PaperMod/`**：它是 submodule，改了不会被仓库记录，主题更新时也会丢。
3. **不要用 `.dark` 写深色样式**：PaperMod 用的是 `html[data-theme="dark"]`。
4. **不要给 Giscus 加 `data-loading="lazy"`**：iframe 初始高度为 0，会永远不加载。
5. **不要随便改专栏目录名**：URL 会变，旧链接需要重定向。
6. **不要对正文图片加 Hugo `.Resize`**：会重新编码，可能破坏原图比例；要压缩就在上传前手动压。
7. **不要在 `draft: true` 的文章里引用不存在的封面**：本地 `-D` 能看到，线上构建后是 404。

---

## 10. 定期维护建议

每月或每季度做一次：

1. 跑一次 `hugo --gc --minify --printPathWarnings`，清掉警告；
2. 检查线上文章图片有没有 404（尤其是新文章）；
3. 检查 Vercel 最近几次构建是否成功；
4. 检查主题是否有更新：

```bash
git submodule update --remote --merge themes/PaperMod
```

更新后重点回归：导航下拉菜单、文章页阅读量、评论区、搜索、背景图。因为 `header.html` 和 `single.html` 是本地复制版，主题更新后如果上游结构变了，需要手动同步这两个文件。

5. 检查 `go.mod` 的 Go 版本是否还能在 Vercel 构建镜像上跑。

---

## 11. 快速命令卡片

```bash
# 新建文章
hugo new content posts/my-post/index.md

# 本地预览（含草稿）
hugo server -D

# 构建检查
hugo --gc --minify --printPathWarnings

# 发布
git add -A
git commit -m "post: 文章标题"
git push

# 查看当前改动
git status --short
git diff --stat
```

---

## 12. 写作小约定（可选）

- 文件名用英文小写短横线：`thinkpad-t430/index.md`，避免中文和空格进 URL；
- 每篇文章至少一个 `description`，一两句话概括；
- 标签控制在 3～6 个，专栏只放一个；
- 图片命名用 `1.webp`、`2.webp` 或语义名，避免 `新建文件`、`IMG_0001`；
- 封面单独命名 `cover.webp`，和正文图片区分开；
- 发布前把文章在浅色和深色模式各看一遍，检查图片、表格、代码块、评论。
