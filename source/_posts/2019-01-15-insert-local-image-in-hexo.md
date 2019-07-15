---
title: Hexo 中完美插入本地图片
date: 2019-01-15 13:54:39
categories:
- Hexo
tags:
- hexo
---

在 Hexo 的官方文档 [资源文件夹章节](https://hexo.io/zh-cn/docs/asset-folders.html) 里，提供了几种插入本地图片的方法。但是，非常遗憾，无论采用哪种方式，在使用其他 Markdown 编辑器进行文章编写时，都无法预览图片。本文介绍了一种方法，可以完美解决此问题。

<!--more-->

## Hexo 官方插入本地图片方法

首先来看官方给出的几种方式。

### 绝对路径

如果你的Hexo项目中只有少量图片，那最简单的方法就是将它们统一放在 `source/images` 文件夹中。然后通过以下方法进行访问。

``` markdown
![](/imges/image.jpg)
```

图片既可以在首页内容中访问到，也可以在文章正文中访问到。

### 相对路径

图片除了可以放在统一的 `source/images` 文件夹中，还可以放在文章自己的目录中。文章的目录可以通过配置  `_config.yml` 来生成。

```
post_asset_folder: true
```

将 `_config.yml` 文件中的配置项 `post_asset_folder` 设为 `true` 后，执行命令 `hexo new post_name`，在 `source/_posts` 中会生成文章  `post_name.md` 和同名文件夹 `post_name`。将图片资源放在  `post_name` 中，文章就可以使用相对路径引用图片资源了。引用图片的方法如下：

``` markdown
![](image.jpg)
```

以上这种引用方法，图片只能在文章中显示，但无法在首页中正常显示。如果希望图片在文章和首页中同时显示，可以使用标签插件语法。

### 标签插件

通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确。在Hexo 2时代，社区创建了很多插件来解决这个问题。但是，随着Hexo 3 的发布，许多新的标签插件被加入到了核心代码中。这使得你可以更简单地在文章中引用你的资源。

引用语法如下：
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

在上述语法下，引用图片的方法如下：

```
{% asset_img example.jpg This is an example image %}
```

通过这种方式，图片将会同时出现在文章和主页以及归档页中。

## hexo-asset-image

无论是以上哪一种方法，在使用其他 Markdown 编辑器进行文章编写时，都无法预览图片。为了解决此问题，可以安装一个图片路径转换的插件 [hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)。先将 `_config.yml` 文件中的配置项 `post_asset_folder` 设为 `true`，然后在根目录下执行以下命令进行插件的安装：

``` bash
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

通过 `hexo new post_name` 命令，在 `source/_posts` 中会自动生成文章 `post_name.md` 和同名文件夹 `post_name`，把所有需要插入的图片放到这个同名文件夹里，通过 Markdown 语法对图片的相对路径引用，即可实现编辑时可预览图片，引用方法如下：

``` markdown
![](post_name/image.jpg)
```

然后在发布时 `hexo-asset-image` 插件会自动将相对路径转为绝对路径，保证在文章和主页以及归档页都能正常显示图片。

但是，这个插件有个小问题，如果网站部署在子目录下，插件生成的图片路径有问题，导致图片无法显示。需要修改插件。插件代码主要保存在 `node_modules/hexo-asset-image/index.js` 这个文件，以下是其中一段代码：

``` js
var link = data.permalink;
var beginPos = getPosition(link, '/', 3) + 1;
// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
var endPos = link.lastIndexOf('/') + 1;
link = link.substring(beginPos, endPos);
```

根据文章链接截取得到 link，其中文章链接一般都是 `http://host:port/yyyy/mm/dd/post_name`，从第三个 `/` 后面取到的就是资源的绝对路径，然后对图片的 src 属性进行处理，去掉文章同名文件夹目录，仅保留图片名，然后与前面获取到的资源绝对路径进行拼接，得到图片的绝对路径。

如果网站部署在子目录下，文章链接是这样的：
`http://host:port/subdirectory/yyyy/mm/dd/post_name`
这时应该从第四个 `/` 后面开始截取。因此，将代码修改如下：

``` js
var link = data.permalink;
var rootArray = config.root.split('/');
var beginPos = getPosition(link, '/', 2 + rootArray.length - 1) + 1;
// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
var endPos = link.lastIndexOf('/') + 1;
link = link.substring(beginPos, endPos);
```

为了使修改后的插件生效，需要先执行 `hexo clean` 清理一下，然后再执行 `hexo g` 重新生成即可。

## 小结

本文介绍了 Hexo 官方插入本地图片的三种方法，但是不推荐使用，因为在使用其他 Markdown 编辑器进行文章编写时，无法预览图片。为了解决此问题，可以安装一个图片路径转换的插件 `hexo-asset-image`，在生成网站时自动将图片相对路径转为绝对路径，使得在文章和主页以及归档页都能正常显示图片。最后，通过修改插件代码，解决网站部署在子目录下图片无法显示的问题。
