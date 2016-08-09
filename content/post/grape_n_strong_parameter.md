---
author: hzmangel
date: "2016-08-10T00:07:40+08:00"

categories:
- Happy coding

tags:
- Programming
- Rails
- RubyOnRails
- TIL

title: |
  Rails - Fix ActiveModel::ForbiddenAttributesError in grape
---

最近尝试在 *Grape API* 中创建一个 *ActiveModel* ，但是在使用 `new` 创建的时候发现会报 `ActiveModel::ForbiddenAttributesError` 错误。想了想估计是碰上了 *stronng parameters* 的问题。按照之前的经验，那就是给参数加上一个permit，但是发现在调用 `permit!` 之后，grape的params变成了空字典，从而造成后续的创建出错。在网上找了一圈，结果发现 grape 自己的Github页面上就有说明，如果需要和 Rails 4 一起使用，需要加上 *hashie-forbidden_attributes* gem。

<!--more-->

细看了眼原因，因为在Rails中，默认传递的params类型为 `ActionController::Parameters` ，而在这个类下可以直接调用 `permit` 或 `permit!` 的方法来完成参数设置，同时会提供 `permitted?` 函数来检查参数是否有效。而 grape 默认传入的参数类型是 `Hashie::Mash` ，并没有提供 `permit` 函数。好在 Ruby 支持 *Monkey Patch* 所以新增的这个 gem 在 `Hashie::Mash` 类中加上了对函数 `:permitted?` 的响应，从而解决了这个问题。

