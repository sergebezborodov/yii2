快速入门
===========

Yii 提供了一整套用来简化实现RESTful风格的Web Service服务的API。
特别是，Yii支持以下关于RESTful风格的API：

* 支持 [Active Record](db-active-record.md) 类的通用API的快速原型;
* 涉及的响应格式（在默认情况下支持JSON 和 XML）;
* 支持可选输出字段的 可定制对象序列化；
* 适当的格式的数据采集和验证错误;
* 支持 [HATEOAS](http://en.wikipedia.org/wiki/HATEOAS);
* 有适当HTTP动词检查的高效的路由;
* 内置`OPTIONS`和`HEAD`动词的支持;
* 认证和授权;
* 数据缓存和HTTP缓存;
* 速率限制;


如下， 我们用一个例子来说明如何用最少的编码来建立一套RESTful风格的API。

假设你想通过RESTful风格的API来展示用户数据。用户数据被存储在用户DB表，
你已经创建了 [[yii\db\ActiveRecord|ActiveRecord]] 类 `app\models\User` 来访问该用户数据.


## 创建一个控制器 <a name="creating-controller"></a>

首先，创建一个控制器类 `app\controllers\UserController` 如下，

```php
namespace app\controllers;

use yii\rest\ActiveController;

class UserController extends ActiveController
{
    public $modelClass = 'app\models\User';
}
```

控制器类扩展自 [[yii\rest\ActiveController]]。通过指定 [[yii\rest\ActiveController::modelClass|modelClass]]
作为 `app\models\User`， 控制器就能知道使用哪个模型去获取和处理数据。


## 配置URL规则 <a name="configuring-url-rules"></a>

然后，修改有关在应用程序配置的`urlManager`组件的配置：

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' => 'user'],
    ],
]
```

上面的配置主要是为`user`控制器增加一个URL规则。这样，
用户的数据就能通过美化的URL和有意义的http动词进行访问和操作。


## 尝试 <a name="trying-it-out"></a>

随着以上所做的最小的努力，你已经完成了创建用于访问用户数据
的RESTful风格的API。您所创建的API包括：

* `GET /users`: 逐页列出所有用户；
* `HEAD /users`: 显示用户列表的概要信息；
* `POST /users`: 创建一个新用户；
* `GET /users/123`: 返回用户为123的详细信息;
* `HEAD /users/123`: 显示用户 123 的概述信息;
* `PATCH /users/123` and `PUT /users/123`: 更新用户123;
* `DELETE /users/123`: 删除用户123;
* `OPTIONS /users`: 显示关于末端 `/users` 支持的动词;
* `OPTIONS /users/123`: 显示有关末端 `/users/123` 支持的动词。

> Info: Yii将在末端使用的控制器的名称自动变为复数。

你可以访问你的API用`curl`命令如下，

```
$ curl -i -H "Accept:application/json" "http://localhost/users"

HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
X-Powered-By: PHP/5.4.20
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

[
    {
        "id": 1,
        ...
    },
    {
        "id": 2,
        ...
    },
    ...
]
```

试着改变可接受的内容类型为`application/xml`，你会看到结果以XML格式返回：

```
$ curl -i -H "Accept:application/xml" "http://localhost/users"

HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
X-Powered-By: PHP/5.4.20
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<response>
    <item>
        <id>1</id>
        ...
    </item>
    <item>
        <id>2</id>
        ...
    </item>
    ...
</response>
```

> Tip: 您还可以通过Web浏览器中输入URL `http://localhost/users` 来访问你的API。
  尽管如此，你可能需要一些浏览器插件来发送特定的headers请求。

如你所见， 在headers响应， 有关于总数，页数的信息，等等。
还有一些链接，让您导航到其他页面的数据. 例如， `http://localhost/users?page=2`
会给你的用户数据的下一个页面。

使用 `fields` 和 `expand` 参数， 您也可以指定哪些字段应该包含在结果内。
例如, URL `http://localhost/users?fields=id,email` 将只返回 `id` 和 `email` 字段。


> Info: 您可能已经注意到了 `http://localhost/users` 的结果包括一些敏感字段，
> 例如 `password_hash`, `auth_key`。 你肯定不希望这些出现在你的API结果中。
> 你应该在 [Response Formatting](rest-response-formatting.md) 部分中过滤掉这些字段。


## 总结 <a name="summary"></a>

使用Yii框架的RESTful风格的API， 在控制器的操作中实现API末端， 使用
控制器来组织末端接口为一个单一的资源类型。

从 [[yii\base\Model]] 类扩展的资源被表示为数据模型。
如果你在使用（关系或非关系）数据库，推荐您使用 [[yii\db\ActiveRecord|ActiveRecord]]
来表示资源。

你可以使用 [[yii\rest\UrlRule]] 简化路由到你的API末端。

虽然不是必须的，为了方便维护您的WEB前端和后端，
建议您开发接口作为一个单独的应用程序。

