<h1 align="center"> laravel-response </h1>

![Test](https://github.com/Jiannei/laravel-response/workflows/Test/badge.svg)
[![StyleCI](https://github.styleci.io/repos/316969462/shield?branch=main)](https://github.styleci.io/repos/316969462?branch=main)

**社区讨论**：[教你更优雅地写 API 之「规范响应数据」](https://learnku.com/articles/52784)

## 介绍

`laravel-response` 主要用来统一 API 开发过程中「成功」、「失败」以及「异常」情况下的响应数据格式。

实现过程简单，在原有的 `response()->json()`进行封装，使用时不需要有额外的心理负担。

遵循一定的规范，返回易于理解的 HTTP 状态码，并支持定义 `ResponseCodeEnum` 来满足不同场景下返回描述性的业务操作码。

## 概览

- 统一的数据响应格式，固定包含：`code`、`status`、`data`、`message`、`error`
- 内置 Http 标准状态码支持，同时支持扩展 ResponseCodeEnum 来根据不同业务模块定义响应码
- 响应码 code 对应描述信息 message 支持本地化，支持配置多语言
- 合理地返回 Http 状态码
- 根据 debug 开关，合理返回异常信息、验证异常信息等
- 支持格式化 Laravel 的 `Api Resource`、`Api  Resource Collection`、`Paginator`（简单分页）、`LengthAwarePaginator`（普通分页）、`Eloquent\Model`、`Eloquent\Collection`，以及简单的 `array` 和 `string`等格式数据返回
- 分页数据格式化后的结果与使用 `league/fractal` （DingoApi 使用该扩展进行数据转换）的 transformer 转换后的格式保持一致，也就是说，可以顺滑地从 Laravel Api Resource 切换到 `league/fractal`

## 安装

支持 Laravel 8/Lumen 8 以上版本，自定义业务操作码部分依赖于  [jiannei/laravel-enum](https://github.com/Jiannei/laravel-enum)，需要先进行安装。

```shell
$ composer require jiannei/laravel-enum -vvv
$ composer require jiannei/laravel-response -vvv

```

## 配置

### Laravel

- 发布配置文件

```shell
$ php artisan vendor:publish --provider="Jiannei\Response\Laravel\Providers\LaravelServiceProvider"
```

- 格式化异常响应

在 `app/Exceptions/Handler.php` 中 引入 `use Jiannei\Response\Laravel\Support\Traits\ExceptionTrait;`  引入以后，对于 ajax 请求产生的异常都会进行格式化数据返回。

### Lumen

- 复制配置文件到 `vendor/jiannei/laravel-response/config/response.php`，到 `config/response.php`

```bash
cp vendor/jiannei/laravel-response/config/response.php config/response.php
```

- 加载配置

```php
// bootstrap/app.php
$app->configure('response');
```

- 格式化异常响应

在 `app/Exceptions/Handler.php` 中 引入 `use Jiannei\Response\Laravel\Support\Traits\ExceptionTrait;`

在 `app/Http/Controllers/Controller.php` 中引入 `use Jiannei\Response\Laravel\Support\Traits\ExceptionTrait;`

- 注册服务容器

```php
$app->register(\Jiannei\Response\Laravel\Providers\LumenServiceProvider::class);
```

## 使用

扩展包本身提供了丰富的单元测试用例[tests](https://github.com/Jiannei/laravel-response/tree/main/tests) ，也可以直接查看测试用例。

或者查看相应的模板项目:

Laravel 版本 Api 开发初始化项目：[laravel-api-starter](https://github.com/Jiannei/laravel-api-starter)

Lumen 版本 Api 开发初始化项目：[lumen-api-starter](https://github.com/Jiannei/lumen-api-starter)

### 成功响应

- 示例代码

```php
<?php
public function index()
{
    $users = User::all();

    return Response::success(new UserCollection($users));
}

public function paginate()
{
    $users = User::paginate(5);

    return Response::success(new UserCollection($users));
}

public function simplePaginate()
{
    $users = User::simplePaginate(5);

    return Response::success(new UserCollection($users));
}

public function item()
{
    $user = User::first();

    return Response::success(new UserResource($user));
}

public function array()
{
    return Response::success([
        'name' => 'Jiannel',
        'email' => 'longjian.huang@foxmail.com'
    ],'', ResponseCodeEnum::SERVICE_REGISTER_SUCCESS);
}
```

- 返回全部数据

```php
{
    "status": "success",
    "code": 200,
    "message": "操作成功",
    "data": [
        {
            "nickname": "Joaquin Ondricka",
            "email": "lowe.chaim@example.org"
        },
        {
            "nickname": "Jermain D'Amore",
            "email": "reanna.marks@example.com"
        },
        {
            "nickname": "Erich Moore",
            "email": "ernestine.koch@example.org"
        }
    ],
    "error": {}
}
```

- 分页数据（支持自定义内层 data 字段名称，比如 rows、list）

```php
{
    "status": "success",
    "code": 200,
    "message": "操作成功",
    "data": {
        "data": [
            {
                "nickname": "Joaquin Ondricka",
                "email": "lowe.chaim@example.org"
            },
            {
                "nickname": "Jermain D'Amore",
                "email": "reanna.marks@example.com"
            },
            {
                "nickname": "Erich Moore",
                "email": "ernestine.koch@example.org"
            },
            {
                "nickname": "Eva Quitzon",
                "email": "rgottlieb@example.net"
            },
            {
                "nickname": "Miss Gail Mitchell",
                "email": "kassandra.lueilwitz@example.net"
            }
        ],
        "meta": {
            "pagination": {
                "count": 5,
                "per_page": 5,
                "current_page": 1,
                "total": 12,
                "total_pages": 3,
                "links": {
                    "previous": null,
                    "next": "http://laravel-api.test/api/users/paginate?page=2"
                }
            }
        }
    },
    "error": {}
}
```

- 返回简单分页数据（支持自定义内层 data 字段名称，比如 rows、list）

```php
{
    "status": "success",
    "code": 200,
    "message": "操作成功",
    "data": {
        "data": [
            {
                "nickname": "Joaquin Ondricka",
                "email": "lowe.chaim@example.org"
            },
            {
                "nickname": "Jermain D'Amore",
                "email": "reanna.marks@example.com"
            },
            {
                "nickname": "Erich Moore",
                "email": "ernestine.koch@example.org"
            },
            {
                "nickname": "Eva Quitzon",
                "email": "rgottlieb@example.net"
            },
            {
                "nickname": "Miss Gail Mitchell",
                "email": "kassandra.lueilwitz@example.net"
            }
        ],
        "meta": {
            "pagination": {
                "count": 5,
                "per_page": 5,
                "current_page": 1,
                "links": {
                    "previous": null,
                    "next": "http://laravel-api.test/api/users/simple-paginate?page=2"
                }
            }
        }
    },
    "error": {}
}
```

- 返回单条数据

```php
{
    "status": "success",
    "code": 200,
    "message": "操作成功",
    "data": {
        "nickname": "Joaquin Ondricka",
        "email": "lowe.chaim@example.org"
    },
    "error": {}
}
```

其他快捷方法

```php
Response::accepted();
Response::created();
Response::noContent();
```

### 失败响应

**不指定 meesage**

```php
public function fail()
{
    Response::fail();// 不需要加 return
}
```

- 未配置多语言响应描述，返回数据

```json
{
    "status": "fail",
    "code": 500,
    "message": "Http internal server error",
    "data": {},
    "error": {}
}
```

- 配置多语言描述后，返回数据

```json
{
    "status": "fail",
    "code": 500,
    "message": "操作失败",
    "data": {},
    "error": {}
}
```

**指定 message**

```php
public function fail()
{
    Response::fail('error');// 不需要加 return
}
```

返回数据

```php
{
    "status": "fail",
    "code": 500,
    "message": "error",
    "data": {},
    "error": {}
}
```

**指定 code**

```php
public function fail()
{
    Response::fail('',ResponseCodeEnum::SERVICE_LOGIN_ERROR);
}
```

返回数据

```php
{
    "status": "fail",
    "code": 500102,
    "message": "登录失败",
    "data": {},
    "error": {}
}
```

**其他快捷方法**

```php
Response::errorBadRequest();
Response::errorUnauthorized();
Response::errorForbidden();
Response::errorNotFound();
Response::errorMethodNotAllowed();
Response::errorInternal();
```

### 异常响应

- 表单验证异常

```php
{
    "status": "error",
    "code": 422,
    "message": "验证失败",
    "data": {},
    "error": {
        "email": [
            "The email field is required."
        ]
    }
}
```

- Controller 以外抛出异常返回

可以直接使用 abort 辅助函数直接抛出 HttpException 异常

```php
abort(ResponseCodeEnum::SERVICE_LOGIN_ERROR);

// 返回数据

{
    "status": "fail",
    "code": 500102,
    "message": "登录失败",
    "data": {},
    "error": {}
}
```

- 其他异常

开启 debug

```php
{
    "status": "error",
    "code": 404,
    "message": "Http not found",
    "data": {},
    "error": {
        "message": "",
        "exception": "Symfony\\Component\\HttpKernel\\Exception\\NotFoundHttpException",
        "file": "/home/vagrant/code/laravel-api-starter/vendor/laravel/framework/src/Illuminate/Routing/AbstractRouteCollection.php",
        "line": 43,
        "trace": [
            {
                "file": "/home/vagrant/code/laravel-api-starter/vendor/laravel/framework/src/Illuminate/Routing/RouteCollection.php",
                "line": 162,
                "function": "handleMatchedRoute",
                "class": "Illuminate\\Routing\\AbstractRouteCollection",
                "type": "->"
            },
            {
                "file": "/home/vagrant/code/laravel-api-starter/vendor/laravel/framework/src/Illuminate/Routing/Router.php",
                "line": 646,
                "function": "match",
                "class": "Illuminate\\Routing\\RouteCollection",
                "type": "->"
            },
            ...
        ]
    }
}
```

关闭 debug

```php
{
    "status": "error",
    "code": 404,
    "message": "Http not found",
    "data": {},
    "error": {}
}
```

### 自定义操作码

```php
<?php
namespace App\Repositories\Enums;

use Jiannei\Enum\Laravel\Repositories\Enums\HttpStatusCodeEnum;

class ResponseCodeEnum extends HttpStatusCodeEnum
{
    // 业务操作正确码：1xx、2xx、3xx 开头，后拼接 3 位
    // 200 + 001 => 200001，也就是有 001 ~ 999 个编号可以用来表示业务成功的情况，当然你可以根据实际需求继续增加位数，但必须要求是 200 开头
    // 举个栗子：你可以定义 001 ~ 099 表示系统状态；100 ~ 199 表示授权业务；200 ~ 299 表示用户业务。..
    const SERVICE_REGISTER_SUCCESS = 200101;
    const SERVICE_LOGIN_SUCCESS = 200102;

    // 客户端错误码：400 ~ 499 开头，后拼接 3 位
    const CLIENT_PARAMETER_ERROR = 400001;
    const CLIENT_CREATED_ERROR = 400002;
    const CLIENT_DELETED_ERROR = 400003;

    const CLIENT_VALIDATION_ERROR = 422001; // 表单验证错误

    // 服务端操作错误码：500 ~ 599 开头，后拼接 3 位
    const SYSTEM_ERROR = 500001;
    const SYSTEM_UNAVAILABLE = 500002;
    const SYSTEM_CACHE_CONFIG_ERROR = 500003;
    const SYSTEM_CACHE_MISSED_ERROR = 500004;
    const SYSTEM_CONFIG_ERROR = 500005;

    // 业务操作错误码（外部服务或内部服务调用。..）
    const SERVICE_REGISTER_ERROR = 500101;
    const SERVICE_LOGIN_ERROR = 500102;
}
```

### 本地化操作码描述

```php
<?php
// resources/lang/zh_CN/enums.php
use App\Repositories\Enums\ResponseCodeEnum;

return [
    // 响应状态码
    ResponseCodeEnum::class => [
        // 成功
        ResponseCodeEnum::HTTP_OK => '操作成功', // 自定义 HTTP 状态码返回消息
        ResponseCodeEnum::HTTP_INTERNAL_SERVER_ERROR => '操作失败', // 自定义 HTTP 状态码返回消息
        ResponseCodeEnum::HTTP_UNAUTHORIZED => '授权失败',

        // 业务操作成功
        ResponseCodeEnum::SERVICE_REGISTER_SUCCESS => '注册成功',
        ResponseCodeEnum::SERVICE_LOGIN_SUCCESS => '登录成功',

        // 客户端错误
        ResponseCodeEnum::CLIENT_PARAMETER_ERROR => '参数错误',
        ResponseCodeEnum::CLIENT_CREATED_ERROR => '数据已存在',
        ResponseCodeEnum::CLIENT_DELETED_ERROR => '数据不存在',
        ResponseCodeEnum::CLIENT_VALIDATION_ERROR => '表单验证错误',

        // 服务端错误
        ResponseCodeEnum::SYSTEM_ERROR => '服务器错误',
        ResponseCodeEnum::SYSTEM_UNAVAILABLE => '服务器正在维护，暂不可用',
        ResponseCodeEnum::SYSTEM_CACHE_CONFIG_ERROR => '缓存配置错误',
        ResponseCodeEnum::SYSTEM_CACHE_MISSED_ERROR => '缓存未命中',
        ResponseCodeEnum::SYSTEM_CONFIG_ERROR => '系统配置错误',

        // 业务操作失败：授权业务
        ResponseCodeEnum::SERVICE_REGISTER_ERROR => '注册失败',
        ResponseCodeEnum::SERVICE_LOGIN_ERROR => '登录失败',
    ],
];
```

## License

MIT