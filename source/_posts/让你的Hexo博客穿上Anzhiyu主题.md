---
title: 让你的Hexo博客穿上Anzhiyu主题
tags:
  - Hexo
  - 博客
  - 安知鱼主题
categories:
  - Hexo
abbrlink: 2414c1c1
date: 2026-09-18 16:11:20
cover: 'https://img.liuranblog.top/file/1789735785875_post_2.png'
---
## 1.前言
在 Hexo、Hugo 等静态博客框架流行的今天，主题的选择成为了提升博客个性化和用户体验的重要因素。AnZhiYu（安知鱼）主题 以其清新简洁的 UI 设计、高度自定义能力以及良好的用户体验，受到了许多博主的青睐。AnZhiYu 主题 是一款基于 Hexo 的现代化博客主题，主打 极简设计 和 功能丰富，适用于技术博客、个人随笔等多种场景。其灵感来自 Butterfly 和 Volantis 主题，集成了多种实用功能，让用户能够更方便地搭建高质量博客。
###  1.1.主题主要特点
 -  📌 UI 设计：清新简洁，支持暗黑模式，阅读体验优秀。
 -  ⚙ 高度自定义：提供多种配置选项，如导航栏、侧边栏、文章封面等。
 -  🚀 高性能：优化了加载速度，支持 PJAX，提升页面切换体验。
 -  🌍 SEO 友好：内置了 SEO 优化策略，利于搜索引擎收录。
 -  🛠 集成功能：支持评论系统（Waline、Twikoo、Valine）、音乐播放器（APlayer）等。
 -  🎨 支持个性化主题配置：可更改主色调、背景图片等。
 -  🔗 社交媒体集成：支持多种社交平台展示，如 GitHub、Twitter、Weibo。

## 2.主题安装
在你的博客目录下打开**CMD**或者右键后选择**Open Git Bush here**
### 2.1.通过Github下载
```bash
git clone -b main https://github.com/anzhiyu-c/hexo-theme-anzhiyu.git themes/anzhiyu
```
如遇安装不上可以使用以下url代理安装
```bash
git clone -b main https://ghproxy.com/https://github.com/anzhiyu-c/hexo-theme-anzhiyu.git themes/anzhiyu
```
### 2.2.通过下载源文件
下载 [最新release版本](https://github.com/anzhiyu-c/hexo-theme-anzhiyu/releases)，然后解压到 themes 目录，并将解压出的文件夹重命名为 anzhiyu
### 2.3.通过npm安装
```bash
npm i hexo-theme-anzhiyu
```
> 💡 此方法只支持 Hexo 5.0.0 以上版本 通过 npm 安装并不会在 themes 里生成主题文件夹，而是在 node_modules 里生成

### 2.4.应用主题
下载安装好主题后，打开根目录文件夹内的**_config.yml**文件，找到以下配置项，把主题改为anzhiyu
```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: anzhiyu
```
### 2.5.安装 pug 和 stylus 渲染插件
```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```
### 2.6.覆盖配置
覆盖配置可以使**主题配置**放置在 anzhiyu 目录之外，避免在更新主题时丢失自定义的配置。
通过 Npm 安装主题的用户可忽略，其他用户建议学习使用。
macos/linux 在博客根目录运行
```bash
cp -rf ./themes/anzhiyu/_config.yml ./_config.anzhiyu.yml
```
Windows 系统手动将**/themes/anzhiyu/_config.yml**复制到根目录下并重命名为**_config.anzhiyu.yml**，或使用以下命令
```bash
cp themes/anzhiyu/_config.yml _config.anzhiyu.yml
```
注意：
 - 只要存在于 _config.anzhiyu.yml 的配置都是高优先级，修改原 _config.yml 是无效的。 
 - 每次更新主题可能存在配置变更，请注意更新说明，可能需要手动对 _config.anzhiyu.yml 同步修改。
 - 想查看覆盖配置有没有生效，可以通过 hexo g --debug 查看命令行输出。
 - 如果想将某些配置覆盖为空，注意不要把主键删掉，不然是无法覆盖的
### 2.7 本地预览与部署
依旧三件套
```bash
hexo clean && hexo g && hexo s
```
现在你的主题就已经安装好了，接下来就是对这个主题DIY了

## 3.主题配置
### 3.1.创建必要页面
接下来生成一下标签页和分类页
在终端内分别输入一下命令
```bash
hexo new page tags
hexo new page categories
```
完成后你就可以找到 **/source/tags/index.md** 和 **/source/categories/index.md**两个文件，对其进行修改
tags：
```markdown
---
title: 标签
date: 2021-04-06 12:01:51
type: "tags"
comments: false
top_img: false
---
```
categories：
```markdown
---
title: 分类
date: 2022-02-23 17:56:00
aside: false
top_img: false
type: "categories"
---
```
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/6432634045c13.png)

### 3.2.配置404页面
主题内置了一个简单的 404 页面，可在设置中开启 本地预览时，访问出错的网站是不会跳到 404 页面的。
如需本地预览，请访问 http://localhost:4000/404.html
在主题配置文件**_config.anzhiyu.yml**内配置
```yml
# A simple 404 page
error_404:
  enable: true
  subtitle: "页面没有找到"
  background:
```
示例图如下
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/64326263a9eda.png!blogimg)

### 3.3.基本配置
#### 3.3.1.导航配置
菜单配置决定了网站顶部的导航栏内容，可以根据自己的需求进行调整：
```yml
menu:
  文章:
    隧道: /archives/ || anzhiyu-icon-box-archive
    分类: /categories/ || anzhiyu-icon-shapes
    标签: /tags/ || anzhiyu-icon-tags
  友链:
    友人帐: /link/ || anzhiyu-icon-link
    朋友圈: /fcircle/ || anzhiyu-icon-artstation
    留言板: /comments/ || anzhiyu-icon-envelope
  我的:
    音乐馆: /music/ || anzhiyu-icon-music
    相册集: /album/ || anzhiyu-icon-images
    小空调: /air-conditioner/ || anzhiyu-icon-fan
  关于:
    关于本人: /about/ || anzhiyu-icon-paper-plane
    闲言碎语: /essay/ || anzhiyu-icon-lightbulb
    随便逛逛: javascript:toRandomPost() || anzhiyu-icon-shoe-prints1
```
格式为：**页面名称: 链接地址 || 图标**，其中图标使用的是 AnZhiYu 主题内置的图标库。
#### 3.3.2.导航栏设置
```yml
# nav相关配置
nav:
  enable: false
  travelling: false
  clock: true
  menu:
    - title: 网页
      item:
        - name: 博客
          link: https://hexo.anheyu.com/
          icon: /img/favicon.png
    - title: 项目
      item:
        - name: 安知鱼图床
          link: https://image.anheyu.com/
          icon: https://image.anheyu.com/favicon.ico
```
#### 3.3.3.社交链接配置
在侧边栏展示你的社交媒体链接：
```yml
social:
  Github: https://github.com/你的用户名 || anzhiyu-icon-github
  BiliBili: https://space.bilibili.com/你的ID || anzhiyu-icon-bilibili
  QQ: tencent://Message/?Uin=你的QQ号 || anzhiyu-icon-qq
  微博: https://weibo.com/你的用户名 || anzhiyu-icon-weibo
  知乎: https://www.zhihu.com/people/你的ID || anzhiyu-icon-zhihu
```
#### 3.3.4.头像配置
```yml
avatar:
  img: /img/avatar/Avatar.jpg
  effect: true
```
#### 3.3.5.首页封面配置
配置首页顶部的封面图片和标题：
```yml
banner:
  tips: 个人博客
  title: 我的博客标题
  image: /img/4CB4B9060DB50206C98341C4CD589637.jpg
  link: /
```
#### 3.3.6.网站图标配置
设置网站的 favicon 图标：
```yml
favicon: /favicon.ico
```
#### 3.3.7.个人卡片
个人卡片 hover 后的显示描述，该描述请在侧边栏配置中的aside.card_author.description中修改，支持 html 显示。
卡片顶部的状态配置：
```yml
# 作者卡片 状态
author_status:
  enable: true
  # 可以是任何图片，建议放表情包或者emoji图片，效果都很好，[表情包速查](https://emotion.xiaokang.me/)
  statusImg: "https://bu.dusays.com/2023/08/24/64e6ce9c507bb.png"
  skills:
    - 🤖️ 数码科技爱好者
    - 🔍 分享与热心帮助
    - 🏠 智能家居小能手
    - 🔨 设计开发一条龙
    - 🤝 专修交互与设计
    - 🏃 脚踏实地行动派
    - 🧱 团队小组发动机
    - 💢 壮汉人狠话不多
```
### 3.4.文章内容配置
#### 3.4.1 新建文章
使用以下命令创建新文章：
```bash
hexo new post "文章标题"
```
#### 3.4.2.文章封面
AnZhiYu 主题支持为每篇文章设置封面图片，在文章的 Front Matter 中添加：
```yml
cover: https://your-image-url.jpg
```
#### 3.4.3.文章置顶
在文章的 Front Matter 中添加：
```yml
top: true
```
#### 3.4.4.评论系统
AnZhiYu 主题支持多种评论系统，如 Gitalk、Valine 等。在主题配置文件中进行相应配置，这里不过多阐述了。

## 4.全局配置
### 4.1标签卖萌
在主题配置文件中，可以自行修改
```yml
# 标签卖萌
diytitle:
  enable: true
  leaveTitle: w(ﾟДﾟ)w 不要走！再看看嘛！
  backTitle: ♪(^∇^*)欢迎肥来！
```
### 4.2.字数统计
如果你需要给文章搞上字数统计，需要下面几个步骤
```bash
npm install hexo-wordcount --save
```
完成后修改主题配置文件
```yml
wordcount:
  enable: true
  post_wordcount: true
  min2read: true
  total_wordcount: true
```
### 4.3.定制化的右键菜单
在主题配置文件中开启**rightClickMenu**即可
```yml
# 右键菜单
rightClickMenu:
  enable: true
```
### 4.4.动效控制
```yml
# 动效
dynamicEffect:
  postTopWave: true # 文章顶部波浪效果
  postTopRollZoomInfo: true # 文章顶部滚动时缩放
  pageCommentsRollZoom: true # 非文章页面评论滚动时缩放显示（仅仅Twikoo生效）
```
### 4.5.页面卡片顶部气泡升起效果
```yml
# 页面卡片顶部气泡升起效果
bubble:
  enable: false
```
### 4.6.深色模式粒子效果 canvas
```yml
# 深色模式粒子效果canvas
universe:
  enable: true
```
### 4.7.隐私协议弹窗
该弹窗一个窗口会话只会弹出一次。
```yml
# 隐私协议弹窗
agreementPopup:
  enable: true
  url: /privacy
```
## 5.参考资源
 - [安知鱼主题文档](https://docs.anheyu.com/initall.html)
 - [安知鱼主题仓库](https://github.com/anzhiyu-c/hexo-theme-anzhiyu)

## 6.结语
通过本教程，你应该已经成功安装并配置了 AnZhiYu 主题。这个主题提供了丰富的自定义选项，你可以根据自己的喜好进行进一步的个性化设置。
如果你在配置过程中遇到任何问题，欢迎在评论区留言讨论。祝你的博客之旅愉快！