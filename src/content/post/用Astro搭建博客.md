---
title: 用Astro搭建博客
description: 无描述
publishDate: 2025-10-31
updatedDate: "2025-10-31"
tags: 
- Blog

draft: true
slug: astro-blog
---


之前用Hugo搭建过一个博客，但后面不了了之。之后想用Nikola等工具搭建博客，但因为对于Admonition的不支持而作罢。最近在看到Astro和一个支持Markdown Admonition的模版后又有了搭建博客的想法。

# 安装pnpm
由于使用的模版包含`pnpm-lock.yml`，因此切换到pnpm。

```sh
npm install --global corepack@latest
corepack enable pnpm
```

# 创建博客
Astro给新手提供了非常友好的CLI自动配置工具：

```sh
pnpm create astro@latest
```

根据CLI提示操作即可。Astro会自动创建模版，安装依赖，初始化Git仓库。

# 博客配置
编辑`src/site.config.js`，改`title`和`description`以更改网站的标题。

将里面的一些`en-GB`替换为`zh-CN`以获得部分中文支持[^1]。


## Netlify设置
将`site.config.js`里的`url`改为Netlify网站的链接。之后添加Astro的Netlify支持：

```sh
npx astro add netlify
```

注意这个自动工具会导致`astro.config.js`的缩进混乱。

```toml
# netlify.toml
[build]
command = "npm run build"
publish = "dist"
```

# 从Tiddlywiki导出到Astro Markdown

[^1]: 这个模版对于多语言支持非常烂，使用的原因暂时是因为对于Admonition的支持