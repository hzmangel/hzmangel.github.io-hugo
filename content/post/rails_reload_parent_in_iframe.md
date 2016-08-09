---
author: hzmangel
date: "2016-04-17T12:59:42+08:00"

categories:
- Happy coding

tags:
- Programming
- Rails
- RubyOnRails
- TIL

title: |
  Rails - Reload Parent Page in iframe
---

最近碰到的一个问题，弹出的 iframe 窗口在做完操作并把结果返回给 controller 后，调用 `render` 或 `redirect_to` 时都只会刷新 iframe 中的内容，而不会将整个页面都刷新。尝试在 iframe 中提交表单时关闭 iframe 窗口，但是依然无效。最后发现还是需要借助于 Javascript ，最后的解决方案如下：

<!--more-->

```ruby
class FooController < ApplicationController
  def parent_reload
    render text: "<script>window.parent.location.reload();</script>"
  end

  def parent_redirect_to(url)
    render text: "<script>window.parent.location.href='#{url}';</script>"
  end
end
```

就是直接把 Javascript 以 HTML 的形式返回给浏览器端，浏览器端在收到内容后自动执行其中的脚本。
