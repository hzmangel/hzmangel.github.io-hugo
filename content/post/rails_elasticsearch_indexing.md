---
author: hzmangel
date: "2016-02-23T00:52:42+08:00"

categories:
- Happy coding

tags:
- Programming
- Rails
- RubyOnRails
- StudyLog
- Elasticsearch

title: |
  Integrate Rails with Elasticsearch - Indexing
---

之前写Rails在查找这块一般都是用DB内置的查询，不过上次试了下用Elasticsearch，比之前想像的要简单，记点东西在这吧。

<!--more-->

## 安装Elasticsearch

这块其实没有太多需要说的，两种方法，从官网下编译好的[二进制解压](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)，或者用[操作系统的安装源](https://www.elastic.co/guide/en/elasticsearch/reference/master/setup-repositories.html)。如果是OSX，可以使用 *homebrew* 安装，命令为 `brew install elasticsearch` ，安装后的启动可以使用 `brew info elasticsearch` 查看。

安装并启动服务后，可以用下面的命令查看系统是否成功启动：

```
$ curl -X GET 'http://localhost:9200'
```

如果一切正常，会返回服务器的状态信息。

------

## 配置Rails

### Gem

Elasticsearch提供了两个Gem用于和Rails集成，分别是 `elasticsearch-rails` 和 `elasticsearch-model` ，两个Gem的作用如下：

**TODO**

### Model - Enable indexes in ES

Elasticsearch是一个单独的服务器，所以在使用前需要考虑以下几点：

* 有哪些Model需要被索引
* 每个Model有哪些字段需要被索引
* 如何在数据库中记录更新后同步Elasticsearch的索引

如果不使用Elasticsearch提供的gem，需要手动去完成上面的几个步骤，但是Elasticsearch的Gem提供了一系列方便使用的函数。

对于需要索引的Model，只需要在代码中加上几行即可以调用 Elasticsearch 的功能。下面是一个Sample

```ruby
require 'elasticsearch/model'

class Blog < ActiveRecord::Base
  include Elasticsearch::Model
end
```

在加入了这几行代码后，这个Model就可以通过 `__elasticsearch__` 来调用Elasticsearch的API了。例如：

```ruby
Blog.__elasticsearch__.client.cluster.health
```

至此，只是在Blog中导入了一些Elasticsearch相关的配置，但是还没有将具体的数据导入到Elasticsearch服务器中。可以使用下面的命令导入数据：

```ruby
Blog.__elasticsearch__.create_index! force: true
Blog.__elasticsearch__.refresh_index!
```

由于Elasticsearch的索引是和model分开存放的，所以在每次数据有更新时，需要手动更新所更新文档的索引。如下所示：

```ruby
Blog.first.__elasticsearch__.index_document
```

不过Elasticsearch提供了Callback的方式来处理这些操作，只需要在Model中引入相应的文件即可：

```ruby
include Elasticsearch::Model::Callbacks
```

在加入这行代码后，调用 **一次** `Blog.import` 即可完成下面的几步操作：

* 将 *Blog* 中的数据导入 Elasticsearch 中
* 加入相应的 *callback* 函数，在Model数据有更新时，同步索引

在运行完上述的命令（手动更新或自动索引）后，即可在Elasticsearch服务器中查找到相应的内容了。


### Model - Choose indexed fields

在默认情况下，所有字段都会被索引。但是在实际使用中，并不是所有的字段都需要索引。Elasticsearch提供了一个函数用来指定需要索引的字段以及对应的值。函数的名称是 `as_indexed_json` 。这个函数是在 `Elasticsearch::Model::Serializing` 定义的，如果需要定制，直接在Model中重新定义自己的函数就好。下面是一个例子：

```ruby
require 'elasticsearch/model'

class Blog < ActiveRecord::Base
  include Elasticsearch::Model

  def as_indexed_json(options = {})
    as_json(
      only: [:title, :content]
    )
  end
end
```

在上面的例子中，无论我的Blog记录中还有什么字段，只有 *title* 和 *content* 字段会被放入Elasticsearch的索引中，也只有这两个字段会返回在查询的结果中。除了内置字段外， `elasticsearch-model` 还支持将函数中方法的返回值也加入索引中，如下所示：

```ruby
require 'elasticsearch/model'

class Blog < ActiveRecord::Base include Elasticsearch::Model

  def as_indexed_json(options = {})
    as_json(
      only: [:title, :content],
      methods: [:author_name, :tag_count]
    )
  end
end
```


此时在索引的字段和返回的结果中，会多出 *author_name* 以及 *tag_count* 的返回值。这种方式可以指定关联记录的索引信息。除此之外，还可以在 `include` 中使用嵌套的字段来标明关系记录。例如：

```ruby
require 'elasticsearch/model'

class Blog < ActiveRecord::Base
  include Elasticsearch::Model

  def as_indexed_json(options = {})
    as_json(
      only: [:title, :content],
      include: { author: {only: :name}, tag: {methods: [:count]} }
    )
  end
end
```

至此，基本的索引构建应该差不多了，后续会记一些查询的东西。
