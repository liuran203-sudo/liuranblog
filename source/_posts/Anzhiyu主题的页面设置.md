---
title: Anzhiyu主题的页面设置
tags:
  - Hexo
  - 安知鱼主题
  - 博客
categories:
  - Hexo
cover: 'https://img.liuranblog.top/file/1790255077874_post_3.png'
abbrlink: '75815004'
date: 2026-09-22 00:17:50
---
## 1.前言
Anzhiyu 内置大量特色独立页面，不需要自己手写页面布局，只需要简单生成页面文件、配置数据源、开启导航菜单，就可以实现丰富的站点功能。
本篇一次性配置：**友人帐 (友链)、留言板、我的装备、关于页面、追番页、朋友圈、相册集、闲言碎语（即刻短文）**。
> ⚠️重要提醒：所有 yml 配置，**禁止使用 Tab 制表符，必须用空格缩进，冒号后面保留空格**，否则页面报错空白。
## 2.进阶配置
### 2.1.友人帐页面
友人帐是 Anzhiyu 主题的友链专属页面，支持多种卡片样式、分组管理、站点预览图，同时自带友链提交表单，是技术博客很常用的页面。
#### 2.1.1.配置
**步骤 1：生成友链页面**
在 Hexo 博客**根目录**打开终端（Git Bash / PowerShell 都可以），执行命令：
```bash
hexo new page link
```
执行成功后，在`source`目录会出现`link`文件夹，路径 `source/link/index.md`，打开这个文件。
**步骤 2：修改 index.md 的 Front-matter**
把文件内容替换为下面代码，你可以自行修改标题、日期、页面简介：
```markdown
---
title: link
date: 2020-12-01 22:19:45
type: "link"
---
```
> ✅重点：`type: "link"` 这一行不能删除、不能写错，少引号或者拼写错误，页面不会渲染友链卡片。
**步骤 3：编写友链数据源 link.yml**
在 `source/_data/` 目录新建文件 `link.yml`，所有友链分组、博主信息全部写在这里。
```yml
- class_name: 框架
  flink_style: flexcard
  hundredSuffix: ""
  link_list:
    - name: Hexo
      link: https://hexo.io/zh-tw/
      avatar: https://d33wubrfki0l68.cloudfront.net/6657ba50e702d84afb32fe846bed54fba1a77add/827ae/logo.svg
      descr: 快速、简单且强大的网站框架
    - name: anzhiyu主题
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      siteshot: https://npm.elemecdn.com/anzhiyu-theme-static@1.1.6/img/hexo.anheyu.com.jpg

- class_name: 推荐博客
  flink_style: telescopic
  hundredSuffix: ""
  link_list:
    - name: 安知鱼
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      siteshot: https://npm.elemecdn.com/anzhiyu-theme-static@1.1.6/img/hexo.anheyu.com.jpg
      color: vip
      tag: 技术

- class_name: 小伙伴
  class_desc: 那些人，那些事
  flink_style: anzhiyu
  hundredSuffix: ""
  link_list:
    - name: 安知鱼
      link: https://hexo.anheyu.com/
      avatar: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.4/img/avatar.jpg
      descr: 生活明朗，万物可爱
      recommend: true
```
**字段详细说明**

| 参数 | 是否必填 | 说明 |
|-----|-----|-----|
| class_name | 必填 | 友链分类名 |
| class_desc | 可选 | 友链分类描述 |
| flink_style | 必填 | 卡片渲染样式，可选 `flexcard` / `anzhiyu` / `telescopic` |
| hundredSuffix | 可选 | 头像图片后缀，用于图床统一裁剪，一般留空 |
| link_list | 必填 | 当前分组下所有友链的数组列表 |
| link_list.name | 必填 | 博主昵称 |
| link_list.link | 必填 | 博客主页完整 URL |
| link_list.avatar | 必填 | 头像图片直链，优先使用稳定图床，避免失效 |
| link_list.descr | 必填 | 博客简短介绍文案 |
| link_list.siteshot | 可选 | flexcard、telescopic 模式生效，站点预览截图链接 |
| link_list.recommend	 | 可选 | 设置`true`，卡片右上角出现「荐」标识 |
| link_list.tag | 可选 | 卡片左上角自定义标签文字 |
| link_list.color | 可选 | 标签背景色，支持十六进制色号，快捷值 `vip`、`speed` |

**步骤 4：导航菜单开启友人帐入口**
打开主题配置文件 `_config.anzhiyu.yml`，找到`menu`配置块，添加友人帐条目，注意缩进：
```yml
menu:
  友链:
    友人帐: /link/ || icon-link
```
成果如图所示
![alt text](https://img02.anheyu.com/adminuploads/1/2023/04/09/6432641611b97.png!blogimg)
#### 2.1.2.与数百博主共同进步
这个功能会在友链页面顶部渲染大量博主头像墙，适合友链数量 30 条以上的站点，友链太少会大面积空白。同样在`_config.anzhiyu.yml`添加：
```yml
# 友情链接顶部相关配置
linkPageTop:
  enable: true
  title: 与数百名博主无限进步
  # 添加博主友链的评论自定义格式
  addFriendPlaceholder: "昵称（请勿包含博客等字样）：\n网站地址（要求博客地址，请勿提交个人主页）：\n头像图片url（请提供尽可能清晰的图片，我会上传到我自己的图床）：\n描述：\n站点截图（可选）：\n"
```
### 2.2.关于页面
个人介绍主页，模块丰富：头像、个人简介、技能标签、时间线生涯、兴趣爱好等。
**步骤 1：生成 about 页面**
```bash
hexo new page about
```
编辑 `source/about/index.md`
```markdown
---
title: 关于
date: 2021-03-30 15:57:51
aside: false
top_img: false
background: "#f8f9fe"
comments: false
type: "about"
---
```
> `type: "about"` 是页面渲染识别标识。
**步骤 2：编写数据源 about.yml**
在`source/_data/about.yml`编写全部个人信息，下面是完整示例，你按需修改、删减模块：
```yml
- class_name: 关于页
  subtitle: 生活明朗，万物可爱✨
  avatarImg: https://npm.elemecdn.com/anzhiyu-blog-static@1.0.0/img/avatar.webp
  avatarSkills:
    left:
      - 🤖️ 数码科技爱好者
      - 🔍 分享与热心帮助
      - 🏠 智能家居小能手
      - 🔨 设计开发一条龙
    right:
      - 专修交互与设计 🤝
      - 脚踏实地行动派 🏃
      - 团队小组发动机 🧱
      - 壮汉人狠话不多 💢
  name: 陈志伟
  description: 是一名 前端工程师、学生、独立开发者、博主
  aboutsiteTips:
    tips: 追求
    title1: 源于
    title2: 热爱而去 感受
    word:
      - 学习
      - 生活
      - 程序
      - 体验
  helloAbout: Hello there!
  skillsTips:
    tips: 技能
    title: 开启创造力
  careers:
    tips: 生涯
    title: 无限进步
    list:
      - desc: EDU,软件工程专业
        color: "#357ef5"
      - desc: EDU,软件工程专业
        color: "#357ef5"
      - desc: EDU,软件工程专业
        color: "#357ef5"
    img: https://bu.dusays.com/2023/04/21/644287166329b.png
  statistic:
    link: /archives
    text: 文章隧道
    cover: https://bu.dusays.com/2023/05/01/644f4b037b930.jpg
  map:
    title: 我现在住在
    StrengthenTitle: 中国，长沙市
    background: https://bu.dusays.com/2023/07/05/64a4c61cb20ef.jpg
    backgroundDark: https://bu.dusays.com/2023/07/05/64a4c63495ac5.jpg
  selfInfo:
    selfInfoTips1: 生于
    selfInfoContentYear: 2002
    selfInfoTips2: 湖南信息学院
    selfInfoContent2: 软件工程
    selfInfoTips3: 现在职业
    selfInfoContent3: 大三学生👨‍🎓
  personalities:
    author_name: 执政官
    personality_type: ESFJ-A
    photo_url: https://bu.dusays.com/2023/07/05/64a4c63495ac5.jpg
    personality_img: https://npm.elemecdn.com/anzhiyu-blog@2.0.8/img/svg/ESFJ-A.svg
    name_url: https://www.16personalities.com/ch/esfj-%E4%BA%BA%E6%A0%BC
  maxim:
    maxim_tips: 座右铭
    maxim_top: 生活明朗，
    maxim_bottom: 万物可爱。
  buff:
    buff_tips: 特长
    buff_top: 脑回路新奇的 酸菜鱼
    buff_bottom: 二次元指数 MAX
  game:
    game_tips: 爱好游戏
    game_title: 原神
    game_uid: "UID: 125766904"
    game_bg: https://bu.dusays.com/2023/04/22/64433bf26e25d.webp
  comic:
    comic_tips: 爱好番剧
    comic_title: 追番
    comic_list:
      - name: 约定的梦幻岛
        href: https://www.bilibili.com/bangumi/media/md5267750/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/27/647166c44b414.webp
      - name: 咒术回战
        href: https://www.bilibili.com/bangumi/media/md28229899/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db4398832e.webp
      - name: 紫罗兰永恒花园
        href: https://www.bilibili.com/bangumi/media/md8892/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db43983d99.webp
      - name: 鬼灭之刃
        href: https://www.bilibili.com/bangumi/media/md22718131/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/24/646db439856a0.webp
      - name: JOJO的奇妙冒险 黄金之风
        href: https://www.bilibili.com/bangumi/media/md135652/?spm_id_from=666.25.b_6d656469615f6d6f64756c65.1
        cover: https://bu.dusays.com/2023/05/30/64760e38d651a.webp
  like:
    like_tips: 关注偏好
    like_title: 数码科技
    like_bg: https://bu.dusays.com/2022/12/06/638f5f05ce1f7.jpg
    like_bottom: 手机、电脑软硬件
  music:
    music_tips: 音乐偏好
    music_title: 许嵩、民谣、华语流行
    music_bg: https://p2.music.126.net/Mrg1i7DwcwjWBvQPIMt_Mg==/79164837213438.jpg
    music_link: /music
  reward_list:
    - name: 海阔蓝
      amount: 8.8
      datatime: 2023-03-28
    - name: LK66
      amount: 66.6
      datatime: 2023-03-24
    - name: 张时貳
      amount: 6.6
      datatime: 2023-01-22
    - name: ZeroAf
      amount: 9.9
      datatime: 2022-12-14
    - name: LuckyWangXi
      amount: 6.6
      datatime: 2022-12-14
    - name: 刀中日月长
      amount: 10
      datatime: 2022-11-16
    - name: 鹿啵包
      amount: 10
      datatime: 2022-11-08
    - name: 疾速k
      amount: 50
      datatime: 2022-09-20
    - name: 伴舟先生大霖子
      amount: 4.03
      datatime: 2022-10-27
      suffix: 贝壳
    - name: Magica_0x0
      amount: 3.36
      datatime: 2022-10-07
      suffix: 贝壳
    - name: 名字就是要短像这样
      amount: 3.36
      datatime: 2022-08-25
      suffix: 贝壳
    - name: Leviathan520
      amount: 1.34
      datatime: 2022-08-23
      suffix: 贝壳
    - name: 托马斯
      amount: 10
      datatime: 2022-08-19
    - name: 哇是猫猫欸
      amount: 1.34
      datatime: 2022-08-19
      suffix: 贝壳
```
**步骤 3：导航菜单**
```yml
menu:
  关于:
    关于本人: /about/ || icon-zhifeiji

```
| 参数 | 备选值/类型 | 是否必填 | 说明 |
|-----|-----|-----|-----|
| class_name | 关于页 | 必填 | 页面类 |
| subtitle | string | 必填 | 副标题 |
| avatarImg | url | 必填 | 头像链接 |
| name | string | 必填 | 作者名称 |
| description | string | 必填 | 描述 |
| aboutsiteTips | object | 必填 | 站点关于提示相关配置 |
| aboutsiteTips.tips | 	string | 必填 | 站点关于提示性文字 |
| aboutsiteTips.title1 | string | 必填 | 站点关于标题文字 1 |
| aboutsiteTips.title2 | string | 必填 | 站点关于标题文字 2 |
| aboutsiteTips.word | list | 必填 | 站点关于标题滚动文字 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |
| class_name | 关于页 | 必填 | 页面类 |