---
author: hzmangel
date: "2015-12-07T21:15:45+08:00"

categories:
- Happy coding

tags:
- Programming
- golang
- StudyLog

title: |
  Go generate study
---

最近在折腾用Golang弄DB，定义完了 `struct` 后发现好像没有 ORM 可以把这个 `struct` 给映射到某张表上，所以需要：

* 手动创建表结构，包括折腾表名和数据结构
* 同步代码中的字段和表结构
* 如果要换DB还得再来一次

于是，开始找有没有像 Rails 中一样的生成器。

<!--more-->

最开始是想用一个脚本解析struct，生成相应的代码，但是在查文档的时候发现golang在1.4版本中引入了 [generate](https://golang.org/doc/go1.4#gogenerate) 命令，它可以解析一个文件并生成另一个文件。官方给的说明里面是从yacc的语法生成go文件。想了想觉得它可以完成生成SQL命令，同步struct和SQL的功能，而且如果做了处理的话还可以方便的在DB间切换，那就试试吧。

## 语法

在介绍如何编写前，先介绍下调用的方式。要成功调用一次generate需要两个条件：

* 已经安装的generator
* 在需要处理的代码中加上 `//go:generate`

这样在文件所在的目录下运行 `go generate` 即可根据注释中所指示的方法，调用相应的生成器了。

## 编写

Golang给出的sample是一个字符串相关的东西，具体可以见 [文档页](https://godoc.org/golang.org/x/tools/cmd/stringer)。[另一个页面](http://www.onebigfluke.com/2014/12/generic-programming-go-generate.html) 也有一份具体的实现可以参考。它主要的思路就是用 `generate` 去处理某个 `.go` 文件，然后生成一些新的东西。主要是用内置的 *AST* 解析文件，并生成相应的东西。


## 我的实现

回到最开始的问题，我有一个定义好的struct，需要去生成SQL命令。所以还是需要用到内置的AST解析器的。除此之外也就是一点点的区分类型型，做转换了。代码放在 https://github.com/hzmangel/struct2schema 这里，README还在编写中。目前代码支持将大部分Go内置的类型转换为SQL类型，不过可能还是有一些不支持的需要手动操作。在DB支持上，目前只看了sqlite3和MySQL。下一步要做的事情估计有这些：

* 支持更多的DB类型，如PostgreSQL， MSSQL 等（Mongo这种无结构的就不需要了......）
* 现在生成的SQL命令中，字段名和Go中的变量名一致，都是Camel String，拟在解析的时候将其变为下划线小写的方式来生成SQL命令（考虑从后面的json字段获取小写名）
* 支持更多的Go数据类型

目前生成出来的代码会直接打印到屏幕上，格式啥的，据说1.6上对模板会有一个更新， [讨论在此](https://github.com/golang/go/issues/9969) ，拭目以待。


## 后记

在写这篇文章的时候，突然反应过来这个 `go generate` 就是会调用在注释中写的命令，根据给定的参数处理。所以理论上说可以给定任何文件，所以还是有不少可玩性的。


