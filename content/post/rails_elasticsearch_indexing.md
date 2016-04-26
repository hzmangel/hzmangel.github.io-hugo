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

之前写Rails在查找这块一般都是用DB内置的查询，不过上次试了下用 Elasticsearch ，比之前想像的要简单，记点东西在这吧。

这篇东西会包含下面几项内容：

* 安装 Elasticsearch
* 关联 Rails 与 Elasticsearch
* 配置索引内容
* Custom Analyzer

本文没有完整系统的介绍，更多的只是一些使用技巧。详细说明请参见[官方文档]()。

<!--more-->

## 安装Elasticsearch

两种方法，从官网下编译好的[二进制解压](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)，或者用[操作系统的安装源](https://www.elastic.co/guide/en/elasticsearch/reference/master/setup-repositories.html)。如果是OSX，可以使用 *homebrew* 安装，命令为 `brew install elasticsearch` ，安装后的启动可以使用 `brew info elasticsearch` 查看。

安装并启动服务后，可以用下面的命令查看系统是否成功启动：

```
$ curl -X GET 'http://localhost:9200'
```

如果一切正常，会返回服务器的状态信息。

------

## 关联 Rails 与 Elasticsearch

### Gem

Elasticsearch提供了两个Gem用于和Rails集成，分别是 [*elasticsearch-rails*](https://github.com/elastic/elasticsearch-rails) 和 [*elasticsearch-model*](https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model) 。其中， *elasticsearch-rails* 为 Rails 的模块加入了 Elasticsearch 的功能，而 *elasticsearch-model* 则为 Ruby 的类提供了一些简化的连接 *Elasticsearch* 的方法，使得相应的类可以通过 `__elasticsearch__` 直接访问 *Elasticsearch* 服务器的资源。

### Model - Enable indexes in Elasticsearch

由于 Elasticsearch 是一个单独的服务器，所以在使用前需要考虑以下几点：

* 有哪些Model需要被索引
* 每个Model有哪些字段需要被索引
* 如何在数据库中记录更新后同步 Elasticsearch 的索引

如果不使用 Elasticsearch 提供的 gem，需要手动去完成上面的几个步骤，但是上面提到的两个 gem 提供了一系列方便使用的函数。代码的东西对着说比较好弄，所以这里有一个 [Demo Project]()，对着来说吧。

这个app就是一个简单的blog，由于只是为了介绍 Elasticsearch ，所以只引入了最简单的部分。

在 `app/models/blog.rb` 中可以看到，需要索引一个 model 需要三步：

1. 在代码中加入 Elasticsearch 的模块。
1. 设置需要索引的模块。
1. 导入数据至 Elasticsearch 。

对于需要索引的Model，只需要在代码中加上几行即可以调用 Elasticsearch 的功能。可以在 `app/models/blog.rb` 中看到下面的代码：

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
Blog.import force: true
```

或者逐步完成：

```ruby
Blog.__elasticsearch__.create_index! force: true
Blog.__elasticsearch__.refresh_index!
```

但是由于 Elasticsearch 的索引是和model分开存放的，所以在每次数据有更新时，需要手动更新所更新文档的索引。如下所示：

```ruby
Blog.first.__elasticsearch__.index_document
```

不过Elasticsearch提供了Callback的方式来处理这些操作，只需要在Model中加入如下代码：

```ruby
include Elasticsearch::Model::Callbacks
```

即可在每次数据有更新时，自动更新索引。在Gem的文档中还有一些更详细的用法，如异步索引，自定义索引等供查阅。

在数据导入完成后即可尝试在 Elasticsearch 查找导入的数据了：

```ruby
Blog.search('*').records.records
```


### Model - Choose indexed fields

默认情况下，所有字段都会被加入 Elasticsearch 的索引。但是在实际使用情况中，可能会碰到只索引某些数据的场景。 Elasticsearch 提供了一个函数用来指定需要索引的字段以及对应的值。函数的名称是 `as_indexed_json` 。这个函数是在 `Elasticsearch::Model::Serializing` 定义的，如果需要定制，直接在Model中重新定义自己的函数就好。如下所示：

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

在上面的例子中，无论我的Blog记录中还有什么字段，只有 *title* 和 *content* 字段会被放入 Elasticsearch 的索引中，也只有这两个字段会返回在查询的结果中。除了内置字段外，还支持将函数中方法的返回值也加入索引中，如下所示：

```ruby
require 'elasticsearch/model'

class Blog < ActiveRecord::Base
  include Elasticsearch::Model

  def as_indexed_json(options = {})
    as_json(
      only: [:title, :content],
      methods: [:author_name, :tag_count],
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

至此，基本的索引构建应该差不多了，在调用过 *import* 后，就可以用基本的查询功能去查找了。


## *Analyzer* 和 *Mapping*

### *Analyzer*

对每一条被索引的内容， ES 都会通过 *Analyzer* 把内容分词后放入服务器（因为ES中是使用 [反向索引](https://zh.wikipedia.org/wiki/%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95) 来查找的，而反向索引的 *token* 就是输入文字经过 *Analyzer* 处理的结果）。而在查询时，如果是全文检索，那么 ES 会将和分析内容时使用的相同的 *Analyzer* 应用于检索词上，并将分词后的检索词用于查找以提高查找的准确性。而如果查询的类型是精确匹配， ES 将不会处理检索词。

根据 [文档](https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html) 的介绍，每一个 *Analyzer* 由 **3** 步组成，分别是：

* *Character filters* : 对输入的内容按字符处理，包含有 *Mapping Char Filter*, *HTML Strip Char Filter* 和 *Pattern Replace Char Filter* ，具体介绍请参阅 [文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)
* *Tokenizer* : 分词器，根据不同的规则，将输入内容分为不同的 token ，[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)
* *Token filters* : 应用于上面分割开的 *token* 上的 filter。 [文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html)

ES 还提供了一些内置的 *Analyzer* ，可以在 [这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) 查到。由于 Rails 中的代码实例会牵涉到 *Mapping* 相关的内容，所以会在下一节中贴代码。


### *Mapping*

对于输入的文本内容，ES会默认将此字段映射为 *string* 类型，并使用 *standard analyzer* ，如果需要一些特殊处理，就需要引入 *Mapping* 了。 *Mapping* 的作用是告诉 ES 某个字段需要按什么样的规则处理。
