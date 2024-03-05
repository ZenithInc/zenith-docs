# Laravel 快速入门

这篇文档对 Laravel 的基础内容进行快速的介绍。介绍了配置、路由、控制器以及模型。了解这些，已经能写出一些小的 Demo 了。

## 基本的配置 {id="configure"}

在开始之前，我们需要对框架进行一些基础的配置，比如说时区以及数据库。 Laravel 的配置都存储在 `config` 目录下，当我们需要配置时区的时候，可以在 `app.php` 中修改:
```PHP
<?php

return [
    'timezone' => 'Asia/Shanghai'
];
```
数据库的配置在 `config/database.php` 文件中, 如下:
```PHP
return [
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
        ],
    ]
];
```
上面的配置中，我们发现对于敏感的数据，是通过 `env` 函数去项目根目录下的 `.env` 文件中获取的。
```yaml
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=app
DB_USERNAME=root
DB_PASSWORD=root
```

## 路由 {id="routes"}

路由在 `routes` 目录下, 默认如果是接口请求配置在 `routes/api.php` 中，如果是页面请求则配置在 `routes/web.php` 中。比如下面是一个 `Hello World` 的接口配置以及实现:
```PHP
<?php

use Illuminate\Support\Facades\Route;

Route::get('/greeting', fn() => 'Hello World');
```
当我们访问 `GET http://localhost:8080/api/greeting` 这个接口后，将会返回 `Hello World`。

其他 HTTP 谓词也是类似，比如 `Route::POST`。

## 控制器 {id="controllers"}

我们来写一个简单的控制器，并注册到路由中。创建文件 `App/Http/Controllers/UserController.php`, 代码如下:
```PHP
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

class UserController extends Controller
{
    public function show(int $id): array
    {
        return compact('id');
    }
}
```
这个控制器只有一个名为 `show` 的方法，接受客户端一个名为 `$id` 的参数，并将这个参数返回到客户端。接着我们来注册路由，代码如下:
```PHP
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::get('/user/{id}', [UserController::class, 'show']);
```
当我们访问 `http://localhost:8080/user/1`, 将会返回 `{"id": 1}` 的 JSON。

如果我们要获取更多更复杂的参数，可以改写成如下形式:
```PHP
public function show(Request $request): array
{
    $id = (int)$request->get('id');
    return compact('id');
}
```
> 当然路由也要跟着改变: `Route::get('/user/{id}', [UserController::class, 'show']);`。然后访问 `http://localhost:8080/api/user` 返回是一样的。

## 模型 {id="models"}

Laravel 提供了非常强大的 ORM，可以将数据表映射为 `Model` 类，简化了增删改查等操作以及关联查询。

```PHP
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model {}
```

上面的示例中，我们定义了一个名为 `User` 的模型，基于约定大于配置的思想。ORM 假设我们的表名为 `users`。如果是模型名称为 `OrderDetail`，则表名为 `order_details`。另外，假设主键名为 `id`, 并且每张表都有 `created_at` 以及 `updated_at` 字段。当创建或更新的时候，会自动写入 `datetime` 类型的值。

所以我们除了创建模型类，几乎可以不配置其他任何东西。接下来，我们就利用这个模型类，完成增删改查。

创建模型的示例如下:
```PHP
$user = new User();
$user->username = 'bob';
$user->password = password_hash('password', PASSWORD_DEFAULT);
$user->save();
```

先查询再更新模型的示例如下:
```PHP
$user = User::find(1);
$user->username = 'tom';
$user->save();
```
在 Laravel ORM 中先查询再更新的做法，可以更好地维护数据的一致性，同时话可以利用 ORM 的更多特性，比如说模型事件以及模型关联等。

删除我们现在都建议使用软删，可以在 `UserModel` 中使用 trait 类 `SoftDeletes`，如下示例:
```PHP
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model {
    use SoftDeletes;
}

User::find(1)->delete();
```

## 总结 {id="summary"}

Laravel 的功能非常强大，这篇文档只是一个入门。后续一系列的文章降序继续深入 Laravel 的实践以及原理。