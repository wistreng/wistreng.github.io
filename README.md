# [TeXt Theme](https://github.com/kitian616/jekyll-TeXt-theme)

[![GitHub release](https://img.shields.io/github/release/kitian616/jekyll-TeXt-theme.svg)](https://github.com/kitian616/jekyll-TeXt-theme/releases)
[![license](https://img.shields.io/github/license/kitian616/jekyll-TeXt-theme.svg)](https://github.com/kitian616/jekyll-TeXt-theme/blob/master/LICENSE)

![TeXt Theme](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/TeXt-home.png)

![TeXt Theme Details](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/TeXt-details.png)

TeXt is a succinct theme for blogging.

TeXt 是针对博客的一款简洁的主题，它虽然简洁但并不简单，该主题模仿了 iOS 11 的风格，有大而突出的标题和圆润的按钮和卡片。

## Features

- 响应式
- 分页（[jekyll-paginate](https://github.com/jekyll/jekyll-paginate)）
- 文章目录（[TOC](http://projects.jga.me/toc/)）
- 文章标签
- 阅读次数统计（[LeanCloud](https://leancloud.cn/)）
- Emoji（[Jemoji](https://github.com/jekyll/jemoji)）
- 评论（[Disqus](https://disqus.com/)）
- Google Analytics
- 联系方式设置（Email, Facebook, Twitter, 微博, 知乎……）
- Web 语意化
- 网站图标的自动化工具（[gulp-svg2png](https://www.npmjs.com/package/gulp-svg2png), [gulp-to-ico](https://www.npmjs.com/package/gulp-to-ico)）
- Color Theme
- RSS（[jekyll-feed](https://github.com/jekyll/jekyll-feed)）

下面简要的介绍下使用的方法，当然如果你对 Jekyll 比较了解可以直接看后面的高级部分，这是该主题增加的一些特有功能。

## How To Use

最简单的方法是直接 fork 到你的 GitHub 仓库然后更改其名称为 `<username>.github.io`，稍等一会儿访问 `https://<username>.github.io` 即可看到页面。接下来你可以在线修改 _config.yml 和 md 文件，或者把它 clone 到本地修改后提交。

当然，你可以在 [Releases 页面](https://github.com/kitian616/jekyll-TeXt-theme/releases)下载最新版本源码，或直接 clone 代码到本地。

### 配置

在 _config.yml 文件里按照说明加上你的信息，例如你的名字和联系方式，网站的标题和描述等等。

在 ./about.md 中写上你的简单介绍，例如我叫小明之类的。

### 写博客

使用 Markdown 编写文章，位于 _posts 目录，文件名采用日期 + 标题的形式，形如 `2017-02-02-Very-Long-Title`。

可以在头信息里设置文章的一些基本信息，包括标题、发布时间和标签等。当然，如果你不设置标题和发布时间，系统会使用文件名中的标题和发布时间，具体详见 [Jekyll: 头信息](http://jekyllcn.com/docs/frontmatter/)。当然，该主题在原有的基础上增加了一些属性，这在后面会讲到。

需要注意的是，该主题的文章列表摘要会默认最多 200 个单词（汉字）的内容。若想控制摘要内容，需要在文章中想要显示到的地方加上 `<!--more-->` 行，具体详见 [Jekyll: 文章摘要](http://jekyll.com.cn/docs/posts/#_6)。

### 安装环境（非必须）

请确保你的电脑上安装了 Ruby 开发环境。（ Linux 和 macOS 自带）

首先安装 github-pages，在项目根目录执行 `bundle install` 即可安装。

推荐安装 Node.js 环境，可以获得更好的开发体验。

### 本地服务（非必须）

如果你安装了 Node.js 环境，只需要在项目根目录运行 `npm run dev` 即可启动本地服务。

如果不想安装 Node.js 环境，则需要以下两步：

启动编译服务，在文件改变时会自动重新编译：

```console
bundle exec jekyll build --watch
```

启动静态服务器：

```console
bundle exec jekyll serve -H 0.0.0.0
```

在浏览器中访问 [http://localhost:4000/](http://localhost:4000/) 即可看到页面。

### 部署与提交

推荐部署到 GitHub Pages 上，简单而免费，详见 [Jekyll: GitHub Pages](http://jekyllcn.com/docs/github-pages/)。

如果你是下载或者 clone 的源码，那么你需要在 GitHub 上建立一个 Repository，然后把项目代码 push 到其对应的分支上（如果以 `<username>.github.io` 命名则对应分支为 `master` ，其他的为 `gh-pages`，详见 [Github Pages: Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)）。

当然你也可以部署到到其他地方。

## 高级

### Color Theme

颜色主题位于文件夹 _sass/colors 中，修改 _sass/settings/colors.scss 的 `@import` 路径即可修改主题，默认主题为 default。

| `default` | `dark` | `forest` | `ocean` |
| --- |  --- | --- | --- |
| ![default](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/colors_default.jpg) | ![dark](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/colors_dark.jpg) | ![forest](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/colors_forest.jpg) | ![ocean](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/colors_ocean.jpg) |

更多颜色主题敬请期待。

### 网站图标

该主题自带了一个“银杏叶”图标，你可以把它替换为自己的图标。网站的图标位于根目录的 favicon.ico 和 ./statics/images/logo 目录下。你会看到 logo 目录中有很多的 png 文件和一个 svg 矢量图文件。那些 png 图片实际上就是根据 svg 矢量图生成的不同大小的图片，这些图片是一些场景可能会用到的大图标，像 iOS 和 Android 的固定到屏幕和 Windows 10 的磁贴。

该主题提供了一个自动化脚本能将 svg 矢量图自动生成 favicon 和 png 文件。你所要做的是：

1. 安装 Node.js 环境

2. 在项目根目录执行 `npm i` 命令

3. 替换 ./statics/images/logo 目录下的 logo.svg 文件

4. 执行 `npm run artwork` 命令，此时 favicon 和 png 便会替换为新 logo.svg 生成的文件

当然如果要追求各个尺寸下图标的显示效果，那还得对不同尺寸的图片进行修改和优化。

### 评论系统

在 _config.yml 文件的 `disqus.shortname` 项填上你在 [Disqus](https://disqus.com/) 上为网站建立的 site 对应的 shortname，需要注意的是 Disqus 在大陆是无法直接访问的。

> 注意：使用评论系统必须在文章的头信息中设置 key 值。

### 阅读量统计

在 _config.yml 文件 `leancloud` 的 `app_id`、`app_key`、`app_class` 项分别填上你在 [LeanCloud](https://leancloud.cn) 为网站建立的应用的对应参数。

> 注意：使用阅读量统计必须在文章的头信息中设置 key 值。

### Google Analytics

在 _config.yml 文件的 `ga_tracking_id` 项填上你在 [Google Analytics](https://analytics.google.com) 上为网站建立的媒体资源对应的跟踪 ID。

### Markdown 头信息增强

除了 Jekyll 官方的头信息外，该主题增加了一些头信息。

| 变量名称       | 可选值          | 描述 |
| ---           | ---           | --- |
| key           |               | 评论系统和阅读量统计使用的文章标识符，如果未设置则评论和统计无效 |
| picture_frame | shadow        | 该文章的图片框样式，如果为 `shadow` 则图片带有阴影边框 |
| modify_date   |               | 该文章的修改时间，不影响首页文章排序（`date` 代表发表时间，会影响文章排序） |
| comment       | true/false    | 该文章是否能够评论，默认为 true（当然你也可以通过不设置 key 来实现，但是这样的话统计也失效了） |

### 其他资源

在 _includes/icon/social 目录下有很多的社交产品图标，例如 Behance、Flickr、QQ、微信等，方便修改和使用。

## 示例

在线访问：[Qi's blog](https://tianqi.name/blog/)

示例源码：[https://github.com/kitian616/kitian616.github.io/](https://github.com/kitian616/kitian616.github.io/)