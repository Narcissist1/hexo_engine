---
title: flask restul 使用体会
date: 2016-11-24 10:41:08
tags: flask restful
categories: 技术相关
---

最近开发的项目里使用 flask restful 作为接口框架，统一的编写格式方便于代码的可读性，和可维护性提高。

# 请求参数解析

## 基本参数解析

这里有个关于请求解析的简单例子，看起来像[flask.Request.values]字典里的两个参数：一个 integer 和 一个 string

```py
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate cannot be converted')
parser.add_argument('name')
args = parser.parse_args()
```
<!--more-->

> 注意：
默认的参数类型(type)是一个 unicode string，在 python2 里面是 unicode，在 python3 里面是 str

如果指定 help 值，当解析出现错误的时候，将会返回 help 内容。如果你不指定 help 信息，默认行为将会返回 type error 本身的错误信息。

默认的，参数(arguments)是非必需的(required)。同样的请求中出现的没有在 RequestParser 中指定的参数将会被忽略掉。

同样的，参数(arguments) 在你的 request parser 中声明，但是并未出现在实际的请求中的会被设置为 None。

## 必需参数(Required Arguments)

声明必需参数，只需将 `required=True` 加入到 [add_argument()]

```py
parser.add_argument('name', required=True,
help="Name cannot be blank!")
```

## 多值和列表（Multiple Values & Lists）

如果你希望接受多个值作为一个列表关联到一个 key 上，你可以传入 `action='append'`

```py
parser.add_argument('name', action='append')
```
像这样请求
```
curl http://api.example.com -d "name=bob" -d "name=sue" -d "name=joe"
```
将会返回如下参数
```py
args = parser.parse_args()
args['name']    # ['bob', 'sue', 'joe']
```
## Other Destinations
如果你希望你的参数经过解析后存储在一个不同的名字下，你可以传入`dest`参数。

```py
parser.add_argument('name', dest='public_name')

args = parser.parse_args()
args['public_name']
```

## 参数位置(Argument Locations)

默认的，[RequestParser] 会试着从[flask.Request.values]和[flask.Request.json]解析参数。
在[add_argument()]中使用`location`参数可以指定从其他位置拉取数据。任何[flask.Request]中的变量都可以使用，例如：

```py
# Look only in the POST body
parser.add_argument('name', type=int, location='form')

# Look only in the querystring
parser.add_argument('PageSize', type=int, location='args')

# From the request headers
parser.add_argument('User-Agent', location='headers')

# From http cookies
parser.add_argument('session_id', location='cookies')

# From file uploads
parser.add_argument('picture', type=werkzeug.datastructures.FileStorage, location='files')
```

> 注意
> 只有 location='json' 时使用， type=list。[See this issue for more details](https://github.com/flask-restful/flask-restful/issues/380)

## Multiple Locations

可以通过传入一个 list 制定多个 location:
```py
parser.add_argument('text', location=['headers', 'values'])
```
当指定了多个 location 时，从所有 locations 里面取出的参数将会合并到一个[MultiDict](http://werkzeug.pocoo.org/docs/0.11/datastructures/#werkzeug.datastructures.MultiDict)里，location 列出了最终结果集里面的取值顺序。

如果参数 location 列表里面包括了 [headers](http://flask.pocoo.org/docs/0.11/api/#flask.Request.headers) location, 则参数名字将不再大小写敏感，将必需和它们的 title case 名字相匹配（[str.title()](https://docs.python.org/3/library/stdtypes.html#str.title)), 指定 `location='headers'`(非 list)将会保持大小写敏感。

## 解析继承(Parser Inheritance)

你经常会对每一个资源使用不同的解析器。问题在于很多的解析器需要解析相同的参数。相比于每次重新写相同的参数，你可以写一个父解析器包含所有的共享参数，通过 [copy()](http://flask-restful.readthedocs.io/en/0.3.5/api.html#reqparse.RequestParser.copy) 方法来扩展它。你可以通过 [replace_argument()](http://flask-restful.readthedocs.io/en/0.3.5/api.html#reqparse.RequestParser.replace_argument) 来替换任意参数。或者通过 [remove_argument()](http://flask-restful.readthedocs.io/en/0.3.5/api.html#reqparse.RequestParser.remove_argument) 来删除参数， 例如：

```py
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('foo', type=int)

parser_copy = parser.copy()
parser_copy.add_argument('bar', type=int)

# parser_copy has both 'foo' and 'bar'

parser_copy.replace_argument('foo', required=True, location='json')
# 'foo' is now a required str located in json, not an int as defined
#  by original parser

parser_copy.remove_argument('foo')
# parser_copy no longer has 'foo' argument
```
## 错误处理(Error Handling)

RequestParser 提供的默认处理错误的方式是在第一次错误发生的地方终止程序，这在你需要处理很多参数比较耗时的情况下是有益处的。然而，将错误捆绑在一起返回给客户端总是好的。这个行为可以在 Flask 层面或者在 RequestParser 实例上指定。在 RequestParser 上绑定错误信息，需要传入 bundle_errors，例如：

```py
from flask_restful import reqparse

parser = reqparse.RequestParser(bundle_errors=True)
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

# If a request comes in not containing both 'foo' and 'bar', the error that
# will come back will look something like this.

{
    "message":  {
        "foo": "foo error message",
        "bar": "bar error message"
    }
}

# The default behavior would only return the first error

parser = RequestParser()
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

{
    "message":  {
        "foo": "foo error message"
    }
}
```
在 Flask 层面配置入下：
```py
from flask import Flask

app = Flask(__name__)
app.config['BUNDLE_ERRORS'] = True
```
> 警告 
> BUNDLE_ERRORS 是 global 设置，会覆盖 [RequestParser] 的 bundle_errors 设置。

## 错误信息(Error Messages)

每个参数对应的错误信息可以通过 help 参数指定。

如果没有指定 help 参数，则这个参数对应的错误信息将会是它本身的 type error 的字符串表示。如果指定 help，则错误信息就是 help 的值。

help 可以包含一个插入的 token，{error_msg}，它会被替换为 type error 的字符串。这种方式允许了自定义化的显示原始错误信息：

```py
from flask_restful import reqparse

parser = reqparse.RequestParser() parser.add_argument(

‘foo’, choices=(‘one’, ‘two’), help=’Bad choice: {error_msg}’
)

# If a request comes in with a value of “three” for foo:

{
“message”: {
“foo”: “Bad choice: three is not a valid choice”,
}

}
```


[flask.Request.values]: http://flask.pocoo.org/docs/0.11/api/#flask.Request.values
[add_argument()]: http://flask-restful.readthedocs.io/en/0.3.5/api.html#reqparse.RequestParser.add_argument
[RequestParser]: http://flask-restful.readthedocs.io/en/0.3.5/api.html#reqparse.RequestParser
[flask.Request.json]: http://flask.pocoo.org/docs/0.11/api/#flask.Request.json
[flask.Request]: http://flask.pocoo.org/docs/0.11/api/#flask.Request