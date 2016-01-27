---
author: hzmangel
date: "2016-01-27T23:21:42+08:00"

categories:
- Happy coding

tags:
- Programming
- Rails
- RubyOnRails
- StudyLog

title: |
  Use Different Auth Method In Rails API Controller
---

开发的时候碰到一个问题，Rails的controller是用devise来提供认证的，如果用户在访问时没有有效的cookie就会被转到登录界面。但是在API的时候不能用cookie，所以需要分开做验证。

<!--more-->

基本想法是对于网页端的请求，继续使用devise进行认证，而对于某些同时提供API接口的Action，则使用token来验证。除此之外，对于POST请求，还需要跳过CSRF token的检查。基本上代码就是这样的（为了测试方便，使用了自己定义的`before_action` ，同时设置所有的检查都基于请求中的 *token* 字段。）

```ruby
class WelcomeController < ApplicationController
  before_action :auth_user!, only: [:foo], unless: -> { request.format.json? }
  before_action :auth_by_token!, if: -> { request.format.json? }

  def foo
    render json: params
  end

  def bar
    render json: params
  end

  private

  def auth_user!
    head :forbidden if params[:token] != '24'
  end

  def auth_by_token!
    head :forbidden if params[:token] != '42'
  end
end
```

此时，如果使用浏览器访问 *foo* , `request.format` 为 *text/html* ，此时就会调用 `auth_user!` ，而访问 *bar* 时，即使是使用浏览器，也不会调用 `auth_user!` 。而对于API请求，统一都会调用 `auth_by_token!` 进行邓处理。

下面是一个简单的测试用例

```ruby
require "rails_helper"

RSpec.describe WelcomeController, :type => :controller do
  describe 'GET #foo.html' do
    it 'responds forbidden if not given valid token' do
      get :foo
      expect(response).to have_http_status(:forbidden)
    end

    it 'responds forbidden if given wrong token' do
      get :foo, {token: 42}
      expect(response).to have_http_status(:forbidden)
    end

    # auth_user! is invoked
    it 'responds success if given correct token' do
      get :foo, {token: 24}
      expect(response).to have_http_status(:ok)
    end
  end

  describe 'GET #foo.json' do
    before :each do
      request.env["HTTP_ACCEPT"] = 'application/json'
    end

    it 'responds forbidden if not given valid token' do
      get :foo
      expect(response).to have_http_status(:forbidden)
    end

    it 'responds forbidden if given wrong token' do
      get :foo, {token: 24}
      expect(response).to have_http_status(:forbidden)
    end

    # auth_by_token! is invoked
    it 'responds success if given correct token' do
      get :foo, {token: 42}
      expect(response).to have_http_status(:ok)
    end
  end
end
```

**注意**

在老版本的Rails上，有一个[bug#9703](https://github.com/rails/rails/issues/9703)，就是它的判断会把 if/unless 和 only/except 分开处理。在这种情况下需要根据 [此条评论](https://github.com/rails/rails/issues/9703#issuecomment-15830313) 中的内容对代码做相应改动。
