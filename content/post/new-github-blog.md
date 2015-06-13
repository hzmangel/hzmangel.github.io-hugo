+++
categories = ["杂九杂十^_^"]
comments = true
date = "2015-06-13T11:36:18+08:00"
draft = false
share = true
slug = "new-github-blog"
tags = ["Programming", "blog", "meta"]
title = "Blog搬(zhe)家(teng)记"
+++

其实把blog从WP挪出来的想法很早前就有了，只是由于拖延症的原因一直没去弄。不过最近可能是处于病情的低谷期，所以就动手了。

当初想把blog搬家的主要需求也就下面这些：

* 能用Markdown写。除了写着舒服外，这不是还能装13嘛～
* 之前那些内容能弄过来，折腾来折腾去弄了这么久，之前的东西还是想保留下的。
* 模板稍微好看点（不过这个最后证实了还是需要自己弄，诶）。

<!--more-->

## Hugo

最开始的时候也想着用 [Jekyll](http://jekyllrb.com/) 或者 [Octopress](http://octopress.org/) ，但是在转换的时候，发现有些`Latex`的代码中的行末注释`{%`会被认成模板字符串的开始字符，然后就悲剧的说我找不到另一半了啊啊啊我要去死啊啊啊，然后就没有然后了...... 想着如果自己写一个的话那又不知道要拖到什么时候了，所以在网上找到了 [Hugo](http://gohugo.io/)。所以，其实我选择它的原因也就是它能把我原来所有的东西都显示出来不出错（格式什么的再说哈）。

## wp2hugo

敲定了用的框架，下一步就是挪东西了。Jekyll提供了一个挪东西的Gem， [Jekyll Import](https://github.com/jekyll/jekyll-import) ，在WP上也有导出的插件，但是拿到的导出文件多少有些问题，有的是没有时间，有的是没有转成Markdown，当然有一个最不能忍的就是导出文件的文件名（我的blog里面的标题是中文，所以也就明白是咋回事了吧...）。本来想去看那些代码自己弄，再一想算了，自己写吧，这种小工具应该不会引发拖延症的。所以就有了这么个玩意： [wp2hugo](https://github.com/hzmangel/wp2hugo) 。这货主要干的事情是：

* 解析Wordpress导出的XML文件。
* 将站点信息存到`config.yaml`中。
* 将文章存放到带有元信息的Markdown文件中（使用 [html2text](https://pypi.python.org/pypi/html2text) 将原来的HTML内容转成Markdown ），使用 **post id** 作为文件名。

基本上我目前的需求是可以被满足了，不过有一个问题是现在页面中引用的图像文件还是放在老站上的，考虑是不是在脚本中加一个选项把它给放到某个assets目录中去。

## Twenty Ten

原来的Blog用的模板是WP自带的 [Twenty Ten](https://wordpress.org/themes/twentyten/) ，试了几个Hugo的模板后不喜欢，于是着手把这货给弄过来。由于有一些Hugo的变量，还有就是在port到后面有点烦了所以直接把bootstrap弄进来了，所以也新开了一个 [GitHub repo](https://github.com/hzmangel/hugo-twenty-ten) 放这东西。不过有两个东西想吐槽，一是分页，二是标签云。

### 分页

Hugo内置了分页，就是在页面中加入 `{{ template "_internal/pagination.html" . }}` ，但是它是把所有页面都列出来的啊，然后我就发现我的页面下面放了2行页码。虽然说增加每页的文章数可以解决这个问题，但这也不是啥解决办法啊。提供的少的可怜的计算功能还有犯晕的模板语法也没法很快弄出来那种显示第一页最后一页的链接，高亮当前页并显示当前页前后各X页，其它页用`...`代替的效果，所以最后就直接用上一页下一页了，回头有空的话考虑用那种页面底部加上loading按钮的做法吧。


### 标签云

Hugo中提供获取所有标签和标签下对应文章数的函数，但是对于生成字体大小不同的标签云来说，它没有提供对应的数学函数。最后选择的做法就是用Hugo把标签名称，数目，以及链接地址生成到某个div的data属性中，然后再用javascript取到其中信息，计算，并生成标签云的代码。Hugo输出的模板是这样的： [sidebar.html](https://github.com/hzmangel/hugo-twenty-ten/blob/master/layouts/partials/sidebar.html)：

```
<div class="sidebar-block">
  <h5>标签</h5>
  <div id="tag_cloud" data-tags="{{ range $key, $value := .Site.Taxonomies.tags }} ['{{$key}}', {{len $value}}, '/tags/{{ $key | urlize }}']; {{ end }}" />
</div>
```

最后生成标签云的js是这样的：[index.js](https://github.com/hzmangel/hugo-twenty-ten/blob/master/static/js/index.js)

```javascript
$( document ).ready(function() {
    var tag_list = eval($("#tag_cloud").data('tags').split(';'));
    tag_list.pop();

    var tag_json = [];
    var total_cnt = 0;
    $.each(tag_list, function( idx, val ) {
        foo = eval(val);
        tag_json.push(foo);
        total_cnt += foo[1];
    })
    generate_tag_cloud(tag_json, total_cnt);
});

var generate_tag_cloud = function(tag_json, total_cnt) {
    tag_cloud_str = "<div id='tag_cloud_canvas'>";

    $.each(tag_json, function(idx, val) {
        font_size = 8 + Math.log(val[1])/Math.log(total_cnt) * 18;
        tag_cloud_str += "<a style=\"font-size:" + font_size +"pt\" href=" + val[2] + ">" + val[0] + "</a>&nbsp; ";
    });

    tag_cloud_str += "</div>";

    $("#tag_cloud").append(tag_cloud_str);
}
```

目前差不多就是这样了，偶尔想到啥新的东西再往上加吧，嘿嘿。

