---
title: 用Astro搭建博客
description: 无描述
publishDate: 2025-10-31
updatedDate: "2025-11-01"
tags: 
- Tiddlywiki
- Astro
slug: astro-blog
---


之前用[Hugo](https://gohugo.io/)搭建过一个博客，但后面不了了之。之后想用[Nikola](https://getnikola.com)等工具搭建博客，但因为对于Admonition的不支持而作罢。最近在看到[Astro](https://astro.build)和[一个支持Markdown Admonition的Astro模版](https://github.com/chrismwilliams/astro-theme-cactus)后又有了搭建博客的想法。

# 安装pnpm
由于使用的模版包含`pnpm-lock.yml`，因此切换到[pnpm](https://pnpm.io/zh/)。

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

注意这个自动工具会导致`astro.config.js`的缩进混乱，建议在终端查看更改后自行更改配置文件。

在项目根目录中创建以下文件，以方便Nelify识别。

```toml
# netlify.toml
[build]
command = "npm run build"
publish = "dist"
```

之后在Netlify上直接按提示选仓库创建网站即可。

# 从Tiddlywiki导出到Astro Markdown
由于Tiddlywiki的灵活性，使用Tiddlywiki在导出Markdown条目时自动加上frontmatter非常方便。添加以下标题为`$:/templates/astro-markdown`的模板导出Markdown条目，使Astro能识别Markdown条目的元数据：

```
\rules except dash horizrule
---
title: {{!!title}}
description: {{{ [<currentTiddler>get[description]else[无描述]] }}}
publishDate: {{{ [{!!created}format:date[YYYY-0MM-0DD]] }}}
updatedDate: {{{ [{!!modified}format:date[YYYY-0MM-0DD]addprefix["]addsuffix["]] }}}<!-- 不加双引号会导致解析错误 -->
tags: <$list filter="[enlist{!!tags}] -未完结 -Blog">
- <<currentTiddler>></$list><%if [all[current]contains:tags[未完结]] %><!-- 排除未完结，Blog等标签 -->
draft: true<%endif%><%if [<currentTiddler>has[slug]] %>
slug: {{!!slug}}<%endif%>
---
`
`
<$view field="text" format="text" />
```

再添加这个条目以在导出菜单中显示Astro Markdown导出菜单：

```
\define renderContent()
{{{ $(exportFilter)$ +[limit[1]] ||$:/templates/astro-markdown}}}
\end
\import [subfilter{$:/core/config/GlobalImportFilter}]
<<renderContent>>

<!-- 字段 -->
<!-- tags: $:/tags/Exporter -->
<!-- condition: [<count>compare:lte[1]] -->
<!-- description: Markdown (Astro) -->
<!-- extension: .md -->
```

[^1]: 这个模版对于多语言支持非常烂，使用的原因暂时是因为对于Admonition的支持