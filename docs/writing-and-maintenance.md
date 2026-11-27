# 博客写作与维护手册

> 本文件放在 `docs/` 下，不参与 Hugo 站点构建。
> 最后核对时间：2026-11-27。
> 目标：让作者本人和其他 AI 在尽量少的上下文下，快速理解站点结构、写作流程和不可随意改动的部分。

---

## 0. 快速事实

- 框架：Hugo extended + PaperMod；当前构建版本 0.167.0
- 部署：GitHub `main` → Vercel 自动执行 `hugo --gc --minify`
- 站点语言：中文为默认语言，位于根路径；英文位于 `/en/`
- 英文定位：中文文章的译文，不独立发布英文文章
- 评论：Waline，服务端 `https://waline.inital.top`
- 全站统计：自建 API `https://stats.inital.top/api/hit`
- 背景图：`https://t.alcy.cc/ycy?json`，同一标签页内固定
- 构建产物：`public/`，已被 Git 忽略
- 不需要 Go，也不要重新引入 Hugo Modules；短代码已经内联到 `layouts/shortcodes/`
- 主题 `themes/PaperMod/` 是 Git submodule，不要直接改主题文件
- 本地 `public/` 可能被 `hugo server` 写入 `livereload.js`，不要拿本地 `public/` 当线上产物判断

---

## 1. 站点整体架构

```text
content/                  Markdown 原稿
    ├── posts/            文章，推荐 Page Bundle
    ├── columns/          专栏 taxonomy 的介绍页
    ├── about.md
    ├── archives.md
    ├── search.md
    ├── guestbook.md
    └── sorted/           隐藏的排序页父级
            ↓
layouts/                  Hugo 模板：页面结构、partials、shortcodes
assets/css/extended/      自定义 CSS
i18n/                     自定义界面文案的中英文翻译
static/                   logo、favicon、Safari 图标等静态资源
            ↓
Hugo 构建
            ↓
public/                   HTML/CSS/图片产物
            ↓
Vercel                    部署 https://blog.inital.top/
```

### 1.1 Hugo 的工作方式

- `content/**/*.md` 写内容
- `layouts/**` 决定页面长什么样
- `assets/css/extended/custom.css` 是主要视觉定制文件
- `static/` 下的文件会原样复制到站点根目录
- `hugo --gc --minify` 生成 `public/`
- 浏览器里的页面 = HTML 骨架 + CSS 皮肤 + JavaScript 行为

### 1.2 主要页面类型

| 页面 | 路径 | 模板 |
|---|---|---|
| 首页 | `/`、`/en/` | PaperMod 默认 + `homeInfoParams` |
| 文章列表 | `/posts/`、`/en/posts/` | `layouts/list.html` |
| 文章详情 | `/posts/xxx/`、`/en/posts/xxx/` | `layouts/_default/single.html` |
| 专栏总览 | `/columns/`、`/en/columns/` | `layouts/columns/terms.html` 或主题模板 |
| 标签总览 | `/tags/`、`/en/tags/` | `layouts/taxonomy.html` |
| 归档 | `/archives/`、`/en/archives/` | PaperMod 的 `archives` 布局 |
| 搜索 | `/search/`、`/en/search/` | PaperMod 的 `search` 布局 |
| 留言板 | `/guestbook/`、`/en/guestbook/` | `layouts/_default/guestbook.html` |
| 排序视图 | `/posts/oldest/` 等 | `layouts/list.html` + `content/sorted/` |

---

## 2. 多语言结构

### 2.1 URL 规则

`hugo.yaml` 当前配置：

```yaml
defaultContentLanguage: "zh"
defaultContentLanguageInSubdir: false
```

所以：

```text
中文：/posts/foo/
英文：/en/posts/foo/
```

中文 URL 保持根路径，不额外加 `/zh/`。

### 2.2 文件规则

中文源文保持原来的无后缀文件名：

```text
content/posts/foo.md
```

英文译文新增 `.en.md`：

```text
content/posts/foo.en.md
```

Page Bundle：

```text
content/posts/foo/
├── index.md
├── index.en.md
└── 图片资源
```

同一个路径下的中文和英文文件会自动配对，**不需要强制改中文文件名为 `.zh.md`**。

如果中英文文件名或 slug 不一致，需要在两边都写相同的：

```yaml
translationKey: "posts/foo"
```

> 不要只在英文文件里写 `translationKey`。两边不一致时，Hugo 可能不认为它们是译文，评论/阅读量也不会串通。

### 2.3 英文只做译文

英文文章不独立创作。约定：

- 英文内容必须能找到对应中文源文
- 未翻译完的 `.en.md` 用 `draft: true`
- 不要在英文文件里写语言无关的固定 `url`
- 典型错误：

```yaml
# 错：英文归档会抢占中文 /archives/
url: "/archives/"
```

英文页应该让 Hugo 自动生成 `/en/...`，不要手写和中文相同的 `url`。

### 2.4 语言切换

- 由 `layouts/partials/header.html` 处理
- 使用当前页的 `.Translations`
- 当前页有对应译文才显示语言入口
- 当前页没有译文则隐藏英文入口，避免跳到 404
- 中英文菜单在 `hugo.yaml` 的 `languages.<lang>.menu` 中分别配置
- 语言菜单链接的是当前页译文，不是各语言首页

### 2.5 hreflang / canonical

- PaperMod 的 `head.html` 已通过 `.AllTranslations` 输出 `hreflang`
- 不要重复在 `extend_head.html` 再加一套 `hreflang`
- canonical 由 PaperMod 生成
- sitemap 由 Hugo 生成；`enableRobotsTXT: true` 已开启

### 2.6 中英文数据是否共享

已共享：

- 站点总 PV / UV
- 文章评论
- 文章阅读量
- 列表页文章卡片阅读量
- 留言板评论：中英文都使用 `/guestbook/` 作为 Waline path

仍分开：

- 搜索索引：`/index.json` 和 `/en/index.json` 各管各语言
- 标签、专栏 taxonomy
- 英文和中文的 taxonomy 术语
- 英文页面的搜索 UI/placeholder 仍需单独翻译

---

## 3. 日常写作流程

### 3.1 新建中文文章

推荐 Page Bundle：

```bash
hugo new content posts/my-post/index.md
```

结构：

```text
content/posts/my-post/
├── index.md
├── cover.webp
├── 1.webp
└── 2.webp
```

单文件也可以：

```text
content/posts/my-post.md
```

但图片路径更容易错，非必要不推荐。

### 3.2 中文 Front Matter 模板

`archetypes/default.md` 当前模板：

```yaml
---
date: '2026-11-25T20:00:00+08:00'
draft: true
title: '文章标题'
author: 'iNITAL'
tags: []
# columns: ['iNITAL 的垃圾站']
# cover:
#   image: "cover.webp"
#   alt: ""
#   caption: ""
---
```

常用字段：

| 字段 | 说明 |
|---|---|
| `title` | 标题 |
| `date` | 发布时间，保留 `+08:00` |
| `draft` | `true` 不发布，`false` 发布 |
| `author` | 作者 |
| `tags` | 标签数组 |
| `columns` | 专栏数组，值必须和 `content/columns/` 目录名一致 |
| `description` | 摘要，建议每篇都写 |
| `math` | 需要公式时写 `math: true` |
| `cover` | 封面图 |
| `lastmod` | 需要覆盖自动最后更新时间时才写 |

> 专栏的 key 是 `columns`（复数），配置来自 `taxonomies: category: columns`。

### 3.3 新建英文译文

直接复制中文文件并改名为 `.en.md`：

```bash
cp content/posts/my-post/index.md content/posts/my-post/index.en.md
```

然后：

1. 翻译正文和 front matter 中的标题、摘要、图注
2. 把 `draft` 改成 `true`，翻译完成后再改 `false`
3. 如果文件名和中文不同，两边都加 `translationKey`
4. 不要把英文文件的 `url` 写成中文路径

英文 Front Matter 可以简化为：

```yaml
---
title: 'English Title'
date: '2026-11-25T20:00:00+08:00'
draft: true
author: 'iNITAL'
tags: ['Test']
description: 'English summary'
# translationKey: 'posts/my-post'   # 只有路径/slug 不一致时才需要
---
```

### 3.4 发布

```bash
hugo --gc --minify --printPathWarnings
git add -A
git commit -m "post: 文章标题"
git push
```

Vercel 会重新构建。发布前确认：

- 中文和英文都没有误留 `draft: true`
- 英文页面内容已经翻译，不是中文复制品
- 封面图片存在
- 本地预览过浅色/深色模式

---

## 4. 正文写法

### 4.1 图片

Page Bundle 内直接写文件名：

```markdown
![](1.webp "~ 图注 ~")
```

当前渲染钩子：

`layouts/_default/_markup/render-image.html`

只做两件事：

- 带 `"标题"` 时输出 `<figure>` + `<figcaption>`
- 加 `loading="lazy"` 和 `decoding="async"`

它**不会**：

- 压缩图片
- 生成多尺寸 `srcset`
- 自动修正纵横比

所以正文图片仍然需要上传前手动压缩，建议单张控制在 300KB 以内。

封面图由 PaperMod 处理，可能生成响应式 `srcset`。

### 4.2 数学公式

Front matter：

```yaml
math: true
```

正文：

```markdown
行内：$E = mc^2$

块级：
$$
\int_0^1 x^2 \, dx
$$
```

没有 `math: true` 的页面不会加载 KaTeX。

### 4.3 Mermaid

正文代码块使用 `mermaid` 语言标记即可。

- `layouts/partials/mermaid.html` 先检查页面有没有 `.mermaid` 节点
- 没有节点就不加载 Mermaid
- 主题切换时会重新渲染

### 4.4 网易云音乐 / Bilibili

短代码已经内联，不依赖 Hugo Modules：

```markdown
{{< netease-cloud-music id=22805132 type=song height=80 width=500 title="Masquerade" >}}

{{< bilibili "BV1uhYueJErx" >}}
```

Bilibili 播放器已默认 `autoplay=0`。

### 4.5 Markdown 扩展

当前启用：

```markdown
==高亮==
++插入++
^上标^
~下标~
~~删除~~
```

Goldmark 的 `unsafe` 没有开启，原始 HTML 默认不渲染。

### 4.6 专栏与标签

- 新专栏：新建 `content/columns/专栏名/_index.md`
- 文章引用：`columns: ['专栏名']`
- 标签自动生成 `/tags/`
- 专栏 URL 和目录名有关，上线后尽量不要改名

### 4.7 引用本篇章节（平滑滚动）

短代码：`layouts/shortcodes/section-link.html`

推荐给目标标题固定 ID：

```markdown
## 配置说明 {#config}
```

正文里引用：

```markdown
详见 {{< section-link "#config" "配置说明" >}}。
```

也支持命名参数：

```markdown
详见 {{< section-link anchor="#config" text="配置说明" >}}。
```

行为：

- 生成 `#config` 的本篇锚点链接
- 复用 PaperMod 已有的文章内平滑滚动
- 不会新开标签页
- 目标标题会按 `scroll-margin-top` 落在视口约 34% 的位置
- 尊重 `prefers-reduced-motion`：系统要求减少动态效果时直接定位
- 只用于本篇章节；跨文章仍用普通链接或 `relref`

目标标题没有固定 ID 时，也可以填 Hugo 自动生成的 ID，但以后改标题文字可能失效。

---

## 5. 评论、阅读量与外部服务

### 5.1 Waline 文章评论

相关文件：

- `layouts/partials/comments.html` → 调用 `waline.html`
- `layouts/partials/waline.html`
- `layouts/partials/comment-path.html`

行为：

- 评论区接近视口 600px 时才加载 Waline
- Waline UI 语言按页面语言切换：`zh-CN` / `en-US`
- 文章评论和留言板评论都通过 `path` 共享

### 5.2 评论/阅读量跨语言共享

`layouts/partials/comment-path.html` 的规则：

- 中文页面：使用自己的 `.RelPermalink`
- 英文页面：在 `.Translations` 中找中文源文，使用中文源文的 `.RelPermalink`
- 找不到中文源文时，退回英文自己的路径

例如：

```text
中文：/posts/foo/
英文：/en/posts/foo/
Waline path：两边都是 /posts/foo/
```

注意：

- 这个机制依赖“中英文文件能配对”
- 同路径的 `foo.md` / `foo.en.md` 自动配对
- 文件名或 slug 不同时，两边都要写 `translationKey`
- 不要随意改这个 partial，否则历史评论和阅读量会分家

### 5.3 留言板

`layouts/_default/guestbook.html`：

- 中文 `/guestbook/` 和英文 `/en/guestbook/` 都使用 Waline path `/guestbook/`
- 所以留言板评论共享
- UI 语言仍按页面切换

### 5.4 阅读量

- 文章详情：`single.html` 里用 `pageviewCount`，带 `path` 和 `data-path`
- 列表卡片：`list.html` 里用 `data-path`
- 两者都通过 `comment-path.html` 取共享路径
- Waline 服务端在 `https://waline.inital.top`

### 5.5 全站 PV / UV

`layouts/partials/extend_footer.html`：

- 调 `https://stats.inital.top/api/hit`
- 只在正式域名 `blog.inital.top` 统计
- 本地/预览不会污染线上数据
- 中英文页面都算进同一个站点总量

### 5.6 搜索

- `/search/` 和 `/en/search/` 分别使用各语言的 `index.json`
- 搜索页已设置 `robotsNoIndex: true`
- 英文搜索页的 `placeholder` 目前仍是中文，需要后续翻译
- 搜索索引目前不跨语言共享

---

## 6. 背景图、毛玻璃与页面组件

### 6.1 随机背景

位置：`layouts/partials/extend_head.html`

- 请求 `https://t.alcy.cc/ycy?json`
- URL 写入 `sessionStorage` 的 `blog-bg-url`
- 同一标签页内翻页不变
- 新标签页/关闭浏览器后重新随机
- 接口失败时退回 `https://t.alcy.cc/ycy`

导航栏有：

- 欣赏背景图按钮
- 更换背景图按钮
- 换图时使用 `.bg-transition-layer` 做交叉淡入
- 支持 `prefers-reduced-motion`

### 6.2 毛玻璃

主要位置：`assets/css/extended/custom.css`

- 使用 `--glass-*` 变量
- 深色模式选择器是 `[data-theme="dark"]`，不要写 `.dark`
- 不支持的浏览器有 `@supports not` 和 `--fb-glass` 兜底
- 卡片、页脚、标签、分页、排序菜单、大纲都有玻璃样式

### 6.3 桌面大纲

- `layouts/partials/toc-sidebar.html`
- 仅在 `ShowToc: true` 且目录链接数大于 1 时显示
- 桌面端 `min-width: 1200px`
- 位置随屏幕自适应，当前高亮项固定在面板中上部
- 点击大纲标题会落在视口约 34% 处
- 移动端隐藏，使用文章内的 `<details>` TOC
- 文章标题的 `scroll-margin-top: 34vh` 也在 `custom.css` 中

### 6.4 列表页排序

- `/posts/` 默认发布时间新到旧
- `/posts/oldest/` 发布时间旧到新
- `/posts/updated/` 最后编辑新到旧
- `/posts/updated-oldest/` 最后编辑旧到新
- 英文对应 `/en/posts/...`
- 排序控制：`layouts/partials/sort-control.html`
- 静态排序页：`content/sorted/`
- 不要在 `.en.md` 里写语言无关的固定 `/posts/...` URL

---

## 7. 核心文件地图

| 文件/目录 | 作用 |
|---|---|
| `hugo.yaml` | 站点配置、语言、菜单、参数、Markdown、taxonomies |
| `vercel.json` | Vercel 构建、HUGO_VERSION、缓存头 |
| `content/posts/` | 中文文章与英文译文 |
| `content/columns/` | 专栏介绍页 |
| `content/sorted/` | 隐藏排序页父级 |
| `content/about.md`、`about.en.md` | 关于页 |
| `content/archives.md`、`archives.en.md` | 归档页 |
| `content/search.md`、`search.en.md` | 搜索页 |
| `content/guestbook.md`、`guestbook.en.md` | 留言板 |
| `layouts/list.html` | 列表页、排序、阅读量、分页 |
| `layouts/_default/single.html` | 文章详情、阅读量、语言切换相关 |
| `layouts/_default/guestbook.html` | 留言板 Waline |
| `layouts/taxonomy.html` | 标签总览、数量统计 |
| `layouts/columns/terms.html` | 专栏卡片 |
| `layouts/partials/header.html` | 导航、语言切换、下拉菜单、背景按钮 |
| `layouts/partials/waline.html` | 文章评论区 |
| `layouts/partials/comment-path.html` | 评论/阅读量共享路径 |
| `layouts/partials/toc-sidebar.html` | 桌面大纲 |
| `layouts/partials/sort-control.html` | 排序菜单 |
| `layouts/partials/background-viewer.html` | 全屏背景、换图 |
| `layouts/partials/extend_head.html` | 背景图、KaTeX、head 扩展 |
| `layouts/partials/extend_footer.html` | 页脚统计、运行时间、Mermaid |
| `layouts/shortcodes/` | 内联的 Bilibili/网易云短代码 |
| `layouts/shortcodes/section-link.html` | 本篇章节引用，文章内平滑滚动 |
| `layouts/_default/_markup/render-image.html` | 正文图片懒加载和图注 |
| `i18n/zh.yaml`、`i18n/en.yaml` | 自定义 UI 文案 |
| `assets/css/extended/custom.css` | 全站视觉定制 |
| `static/` | logo、favicon、Safari 图标等 |
| `docs/` | 本手册，不参与构建 |

---

## 8. 部署与本地开发

### 8.1 Vercel

- 推送到 `main`
- Vercel 使用 `HUGO_VERSION=0.167.0`
- 构建命令：`hugo --gc --minify`
- 输出目录：`public`
- `/assets/*` 有长期缓存

### 8.2 本地预览

```bash
hugo server -D
```

- `-D` 包含草稿
- 不要用本地 `public/` 判断线上状态
- 本地 dev server 可能写入 `livereload.js`，这不是线上问题

### 8.3 构建检查

```bash
hugo --gc --minify --printPathWarnings
```

当前构建会剩下 Hugo/PaperMod 自身的弃用警告：

- `languageCode` 配置弃用
- `LanguageDirection` / `LanguageCode` / `LanguageName` 主题用法弃用

这些不是当前构建错误，但主题升级时值得处理。

---

## 9. 已知问题 / 待办

> 以下是 2026-11-25 的状态，不要误认为已经修好。

### 9.1 英文内容还没翻完

这些 `.en.md` 仍包含中文，或只是占位：

- `content/posts/hello-world.en.md`
- `content/about.en.md`
- `content/guestbook.en.md`
- `content/search.en.md`（placeholder 中文）
- `content/columns/iNITAL 的垃圾站/_index.en.md`
- `content/columns/不止 initIal/_index.en.md`
- `content/posts/yoga-3-11/index.en.md`（正文中文，但 `draft: true`）

处理原则：

- 未翻译完：`draft: true`
- 已经上线但不完整：翻译或 `robotsNoIndex`
- 不要为了“让英文站有内容”而保留中文正文

### 9.2 英文专栏封面 404

- `/en/columns/iNITAL 的垃圾站/cover.webp`
- `/columns/iNITAL 的垃圾站/page/2/cover.webp`

原因：`layouts/columns/terms.html` 的兜底相对路径以及英文 branch bundle 资源没有复制到英文输出。修专栏封面时要同时检查分页页和英文页。

### 9.3 一处正文坏链

`content/posts/HUAWEI-MateBook-E-2019/index.md` 中有一处链接缺少 `https://`：

```text
blog.inital.top/posts/huawei-matebook-e-2019-linux/
```

会被浏览器当成相对路径。

### 9.4 正文图片偏大

当前正文图片没有响应式处理，单张可能达到 1 MB 级。建议做 Markdown 图片 render hook，生成多尺寸 WebP。封面图已有 `srcset`，不要和正文图片混淆。

### 9.5 搜索/分类仍按语言分开

- 搜索索引不共享
- 标签、专栏 taxonomy 不共享
- 留言板评论已经共享

如果以后要做到“全站搜索包含中英文”，需要单独合并索引，不是改一个配置就能完成。

### 9.6 测试资源

`static/` 中可能有测试用图片/未使用图片。清理前先全局搜索引用，不要直接删除正在被文章引用的文件。

---

## 10. 不要做的事

1. 不要提交 `public/`
2. 不要直接改 `themes/PaperMod/`
3. 不要把中文文章批量改名为 `.zh.md`，当前约定是无后缀中文 + `.en.md` 英文
4. 不要在 `.en.md` 里写语言无关的固定 `url`
5. 不要只在一侧写 `translationKey`
6. 不要重复添加 `hreflang`
7. 不要随意改 `layouts/partials/comment-path.html`
8. 不要用 `.dark` 写深色样式，使用 `[data-theme="dark"]`
9. 不要把 `search.md` / `search.en.md` 的 `robotsNoIndex` 删掉
10. 不要引入 Hugo Modules 或依赖 Go 的构建步骤
11. 不要把 `layouts/partials/waline.html` 改成非懒加载
12. 不要在 `draft: true` 的文章里引用不存在的封面或图片

---

## 11. 发布 Checklist

- [ ] 中文 `draft` 已按需关闭
- [ ] 英文 `.en.md` 已翻译，不是中文复制品
- [ ] 英文未完成项保持 `draft: true`
- [ ] `title`、`date`、`description` 正确
- [ ] `tags`、`columns` 名称存在
- [ ] 封面和正文图片文件存在
- [ ] 中英文路径能配对，评论/阅读量路径符合预期
- [ ] 浅色/深色模式都预览过
- [ ] `hugo --gc --minify --printPathWarnings` 通过
- [ ] `git status` 没有多余文件
- [ ] push 后确认 Vercel 构建成功
- [ ] 线上检查文章、图片、评论、阅读量

---

## 12. 给其他 AI 的快速交接

如果让其他 AI 改这个站，先让它读：

1. 本文件
2. `hugo.yaml`
3. `layouts/partials/comment-path.html`
4. `layouts/partials/header.html`
5. `layouts/list.html`
6. `layouts/_default/single.html`
7. 与任务相关的 partial / CSS

需要它先确认：

- 当前语言是 zh 默认、en 在 `/en/`
- 英文只做译文，不独立发布
- 评论/阅读量按中文路径共享
- 留言板共享 `/guestbook/`
- 搜索和 taxonomy 仍然分开
- `public/` 不是源码
- 主题 submodule 不直接改

改完后至少跑：

```bash
hugo --gc --minify --printPathWarnings
```

并检查：

- 中文页面是否仍是 `lang="zh"`
- 英文页面是否仍是 `lang="en"`
- 归档 `/archives/` 和 `/en/archives/` 是否都存在
- 文章评论/阅读量的 `data-path` 是否仍指向中文路径
- 搜索页是否仍为 `noindex`
- `robots.txt` 和 `sitemap.xml` 是否还存在

---

## 13. 命令速查

```bash
# 本地预览（含草稿）
hugo server -D

# 只构建检查
hugo --gc --minify --printPathWarnings

# 新文章（Page Bundle）
hugo new content posts/my-post/index.md

# 发布
git add -A
git commit -m "post: 文章标题"
git push

# 查看改动
git status --short
git diff --stat
```
