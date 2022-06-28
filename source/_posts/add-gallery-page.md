---
title: Hexo-Next 搭建相册
toc: true
date: 2022-06-05 23:46:42
tags: [hexo, Next]
categories: ["build blog pages"]
description: 本文记录了hexo相册的搭建过程。
---

# 准备工作

- 本节内容是基于实现本地保存图片功能前提记录的笔记，如果是用图床外链，可以跳过所有涉及到 `img` 文件夹以及 `img/s` 文件夹的步骤。
- 本相册的所有功能均仅测试了在 `hexo-theme-next` 下实现的效果，使用其它主题时部分布局尺寸数据可能会出现偏差，需要自行根据主题设置修改。

# 创建目录

- 在博客根目录输入命令 `hexo new page gallery`；
- 在 `hexo/source/gallery` 目录下建立你需要的分类相册文件夹；
- 进入 `hexo/source/gallery` 目录，新建 `img` 文件夹用来存放相册封面；
- 在每个相册文件夹中创建 `img` 文件夹用来存放大图，以及 `img/s` 文件夹用来存放缩略图。

整个 gallery 目录结构如下：

```bash
./gallery/
├── index.md # 相册主界面文件
├── img 
│   ├── s
│   │   └── sample.jpg # 主界面相册封面缩略图
│   └── sample.jpg
└── sample # sample 相册
    ├── img
    │   ├── s 
    │   │   └── sample.jpg #图片缩略图
    │   └── sample.jpg # 图片地址
    └── index.md #分类相册界面文件
```

# 相册主界面

- 打开 `gallery/index.md`，将 title 设置成你需要的相册页面名称；
- （可选）在日期下方加上 `comments: false` 关闭评论；
- 复制以下代码粘贴至正文，并按需求修改相册描述、相册名、相册文件夹名以及封面图文件名：

```html
<center>！相册描述【此行可删除】</center>
<center>自定义分隔符【此行可删除】</center>
<div class="gallery-page">
	<div class="gallery-list">
		<div class="gallery-column">
			<div class="gallery-item">
				<a href="【！相册文件夹名】"><img src="img/【！封面图文件名】.jpg">
				</a>
				<p>- ！相册1 -</p>
			</div>
			<div class="gallery-item">
				<a href="【！相册文件夹名】"><img src="img/【！封面图文件名】.jpg">
				</a>
				<p>- ！相册2 -</p>
			</div>
		</div>
		<div class="gallery-column">
			<div class="gallery-item">
				<a href="sample"><img src="img/sample.jpg">
				</a>
				<p>- 相册名 -</p>
			</div>
		</div>
		<div class="gallery-column">
			<div class="gallery-item">
				<a href="sample"><img src="img/sample.jpg">
				</a>
				<p>- 相册名 -</p>
			</div>
			</div>
		</div>
	</div>
</div>
<center>自定义分割线【此行可删除】</center>
```

1. 需要使用外链的场合，将 `<img src="img/【封面图文件名】.jpg">` 中的内容替换为图床外链地址即可；
2. 代码中 `<div class="gallery-column">` 元素为分列显示相册的列数，可按需要增减；
3. 新增相册时请确认代码添加在 `<div class="gallery-column">` 元素内部，否则会造成显示错误。

# 分类相册界面

- 打开 `gallery/相册名/index.md`，将 title 设置成你需要的相册页面名称；
- （可选）在日期下方加上 `comments: false` 关闭评论；
- 复制以下代码粘贴至正文，并按需求修改相册描述、图片名以及缩略图文件名：

```html
<center>！相册描述【此行可删除】</center>
<center>自定义分隔符【此行可删除】</center>
<div class="gallery-page">
	<div class="img-list">
		<div class="img-column">
			<a href="img/【！图片名1】.jpg" target="_Blank"><img src="img/s/【！缩略图文件名1】.jpg"></a>
			<a href="img/【！图片名2】.jpg" target="_Blank"><img src="img/s/【！缩略图文件名2】.jpg"></a>
		</div>
		<div class="img-column">
			<a href="img/sample.jpg" target="_Blank"><img src="img/s/sample.jpg"></a>
		</div>
		<div class="img-column">
			<a href="img/sample.jpg" target="_Blank"><img src="img/s/sample.jpg"></a>
	</div>
</div>
<center>自定义分割线【此行可删除】</center>
```

1. 需要使用外链的场合，将 `<img src="img/【图片名】.jpg">` 中的内容替换为图床外链地址即可，如果图床加载速度够快可以用同一个链接填充缩略图部分；
2. 代码中 `<div class="img-column">` 元素为分列显示图片的列数，可按需要增减；
3. 添加图片时请确认代码添加在 `<div class="img-column">` 元素内部，否则会造成显示错误。

# CSS 样式

修改`style` 文件，`vim source/_data/styles.styl` 

```css
/*gallery*/

.gallery-page {
	margin-top: -50px;
}
.img-list,
.gallery-list {
	display: flex;
	flex-direction: row;
	flex-wrap: nowrap;
	align-items: flex-start;
}
.img-column {
	display: flex;
	flex-direction: column-reverse;
}
.img-column a,
.gallery-column a {
	border-bottom: 0px;
}
.gallery-item {
	margin-bottom: -50px
}
.gallery-item p {
	margin: -25px auto -10px;
	max-width: 50%;
	text-align: center;
	font-size: 15px;
	color: $black-deep;
	background: rgba(255,255,255,.3);
	border-radius: 7px;
	border: 1px solid $black-deep;
	box-shadow: 0 8px 20px -8px rgba(0,0,0,.3);
}
.posts-expand .post-body .gallery-column a img {
	height: 250px;
	width: 300px;
	object-fit: cover;
}
@media (max-width: 767px){
	.gallery-item p {
		min-width: 75px;
		font-size: 13px;
	}
}
```

1. `@media` 标签内的样式是防止移动端浏览时相册名被强制换行的，建议保留；
2. 其余样式除了 flex 相关行与 object-fit 样式以外，均可根据需要自行更改，在此不作赘述。

# 一些好看的Blog相册

[- 画廊 - | †少女癌† (co5.me)](https://co5.me/gallery/)

[Galleries – ✏Victor's Blog (jiahongw.com)](https://hugo.jiahongw.com/zh/gallery/)

[Gallery | Blog of Tony Yuan (yuan901202.github.io)](https://yuan901202.github.io/gallery/)

[光影集 | 频率 (pinlyu.com)](https://pinlyu.com/album/)

[Gallery | 鱼鱼书 ❤ 菜子 (yuyushu.online)](https://yuyushu.online/gallery/)

[我的相册 - 银河小徐 (hasaik.com)](https://hasaik.com/photos/)

[辣椒の酱 (removeif.github.io)](https://removeif.github.io/album/)

[画廊 | 小丁的个人博客 (tding.top)](https://tding.top/gallery/)

# 参考文献

[1] [css+markdown 实现 hexo 相册【进阶篇】](https://co5.me/2018/181112-gallerry2.html)

[2] [30 分钟学会 Flex 布局 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/25303493)

