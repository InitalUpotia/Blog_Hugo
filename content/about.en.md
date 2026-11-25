---
date: '2026-10-09T20:26:34+08:00'
draft: false
title: 'About'
author: 'iNITAL'
cover:
    image: "about.webp"
    alt: "A sample cover image"
    caption: "~ Obligatory Sakuya to start ~"
---

==**Caution: This page is not fully translated.**==

## About the site

This is my personal blog site. Here's roughly what I plan to cover:

- **ACGN**:

  Share my reviews of the anime I've watched, the games I've played, and the comics I've read. (You can think of it as an extension of X.)

  I will also share some great fanworks, pretty pictures and memes from time to time.

- **Technology**:

  Share some useless tips through tinkering. After all, even this site was built entirely under deepseek's guidance.

  If you want to see some strange gadgets, you can go to [iNITAL's Retro Station](https://blog.inital.top/columns/inital-%E7%9A%84%E5%9E%83%E5%9C%BE%E7%AB%99/) to have a look.

- **Life**:

  Share what I saw and felt during my travels, or some amusing trivia. I will also post some thoughts and hot takes on trending topics online.

- **Study**:

  Honestly, I don't think I'd ever write this kind of thing for fun~

  It's more of a memo pad, really.

## About me

A university student of, shall we say, complex composition.

I have a talent for writing excruciatingly detailed, blow-by-blow posts.

My social media:
- [Bilibili](https://space.bilibili.com/1589490577)
- [Zhihu](https://www.zhihu.com/people/loner-21-67)
- [Bangumi](https://chii.in/107775)

## About the posts

If you find anything wrong, feel free to point it out in the comments, or DM me on social media. I'll fix it as soon as possible.

## Update log

### 2026-10-09: Establishment
- Built the site, bought the domain, and ran the test.

### 2026-10-10: Added features & fixed bugs
- ~~Supported Giscus comments.~~ (Now abandoned; switched to Waline)
- Supported KaTeX.
- Supported some extra Markdown features.
- Supported HugoMods shortcodes. NetEase Cloud Music and Bilibili links are now available.
- ~~Supported Vercount to display view counts.~~ (Now abandoned; switched to Waline)
- Fixed an issue where Chinese characters were excluded from the word count.

### 2026-10-11: Added features
- Added a menu with aero style. (Meanwhile, `Links` was renamed to `EGO Complex`)

### 2026-10-18: Added features
- Supported image captions.
- Supported automatic first-line indent.

### 2026-10-19: Added features
- Supported search.

### 2026-10-24: Changed features
- Renamed `Categories` to `Columns`, and changed its initial style. (From a tag-like layout to an `image + title` list, just like the columns on Zhihu)
- Created my first column: [iNITAL's Retro Station](https://blog.inital.top/columns/inital-%E7%9A%84%E5%9E%83%E5%9C%BE%E7%AB%99/).

### 2026-11-03: Added features
- Supported the display of `Last updated`.

### 2026-11-04: Added a new column & fixed bugs
- Created the new column: [Beyond initIal](https://blog.inital.top/columns/).
- Fixed an issue where column thumbnails were displayed incorrectly.
- Fixed an issue where column names defaulted to a capitalized first letter. (Actually, that's a Hugo feature, but it can't display names like *iNITAL*, so I dropped it)
- Refactored the column framework, and added brief descriptions for the columns.
- Changed the font, and fixed the issue where `l`, `1` and `I` couldn't be clearly identified.

### **2026-11-08: Added features & fixed bugs & improved performance**

Thanks to the super power of deepseek-v4-flash-1017, the previous trash codes were improved, and some features were added.

- New features:
  - Added a counter for the number of days the site has been running.
  - Added post pagination.
  - **[Liciyuan API](https://t.alcy.cc/) was connected, it will randomly update the background image.**
  - The site was fully applied with the aero style.
  - **Waline was connected. Added [Guestbook](https://blog.inital.top/guestbook/), with support for anonymous messages.**
- Fixed bugs:
  - Fixed an issue where the inline KaTeX didn't work.
  - ~~Fixed an issue where Giscus's color wouldn't change in dark mode.~~
  - Merged the two footers.
- Performance improvements:
  - KaTeX now renders selectively instead of globally.
  - Added `.gitignore` so temporary build files are no longer published.
  - Cleaned up a large amount of duplicated, unused CSS.

I think the feature additions have wrapped up for now. Pushing a lightweight Hugo + PaperMod setup this close to the flashiness of WordPress themes is already the limit.

Sure, I'd still like to cram a lot more stuff and effects in. But the server is overseas now, so loading speed is hard to guarantee. And anyway, for a blog, content is what matters most, isn't it?

### 2026-11-11: Added features & fixed bugs
- Supported Mermaid diagrams.
- Fixed an issue where the mobile navigation bar was not fully displayed and had no aero effect.

### 2026-11-15: Fixed bugs & changed features
- Post view counts now use Waline instead of Vercount. The numbers are more accurate, the cache-overwrite bug is gone, and it loads faster. (As a result, previous view counts were reset to zero)
- Slightly reworked the list page; it now displays view counts too.

### 2026-11-18: Fixed bugs & changed features
- Site-wide visitor counts also dropped Vercount and switched to a self-built API. More accurate, and faster to load. (Previous counts were reset to zero)
- Post comments dropped Giscus and now fully use Waline. (Previous comments were cleared — though there were barely any)

### 2026-11-23: Fixed bugs & changed features & added features
- Reworked the previously counterintuitive "Next page" pagination display logic to be more intuitive.
- Added a `View background image` button to the left of the navigation bar. Now you can download your loved waifu!
- Changed the desktop post list to a card layout similar to the previous `Columns` style.
- Applied aero effects to every tag on the `Tags` page, and added hover effects. (Consistent with the tag style in posts)
- Fixed an issue where older browsers without `backdrop-filter` (e.g. MyPal) couldn't correctly display the aero effect. (Personally tested: the site now works perfectly in MyPal on Windows XP SP3 32-bit.)
- Fixed an issue where NetEase Cloud Music links failed to load on mobile.

### 2026-11-24: Added features & fixed bugs
- Added `Sorting` to the `Posts` page, supporting sorting by `Published date` and `Last updated`. (I wanted to sort by view count too, but that data comes from Waline and would require fetching everything at once, which would be very slow. For now only the `Posts` page can be sorted; it's doable for `Tags` and `Columns` subpages, but it would mean abandoning most of PaperMod's features — at that point a different solution would be better.)
- Added an outline on the left side on desktop.
- Added a `Change Background Image` button to the left of the navigation bar.
- Fixed an issue where Bilibili videos would autoplay.

Honestly, I keep thinking it would have been better to start with Hexo instead of Hugo. All this time I've mostly been chasing the look of Hexo's Butterfly theme, and Hexo themes have been heavily modded by countless self-hosted bloggers — a lot of the time I wouldn't have had to build my own wheels with Deepseek. As for Hugo's fast loading, you won't really feel it until you have 30+ posts.

That said, Hugo is much easier when it comes to writing and maintenance. If you switch devices, just pull the repo and push it back — no environment setup like Hexo requires.

### 2026-11-25: Added features
- Displayed `post count` and `tag count` on the right side of the `Posts` page and `Tags` page.
- **The site now supports English**, more content will be translated and moved over gradually.
- Added an avatar to the browser tab. (Finally got this one in)
