# 在 Laravel 中进行单元测试

单元测试是提升软件质量的重要手段之一，这篇文档将会深入 Laravel 框架的单元测试，它提供了开箱即用的测试工具。

## 概述 {id="feature-and-unit"}

在 Laravel 的测试框架中，**有两种主要类型的测试：Feature 测试和 Unit 测试**。这两种测试方式各有特点和适用的场景：

* **Unit 测试:** 通常针对程序中的一个单独的 "单元"（例如，一个函数或者一个方法）进行测试。
  * 它们通常会模拟出或者隔离开这个单元的依赖关系，因此这种测试通常运行得很快。
  * Unit 测试可以帮助开发者保证每个单独的函数或者方法都在预期的条件下正常工作。它们的作用就好像程序的 "安全网"，以便在更改或者重构代码的时候，可以及时发现可能引入的错误。
  * Laravel 中，Unit 测试通常放置在 `tests/Unit` 目录下
  * 不应该在这些测试用引入数据库、中间件或第三方系统，要 Mock 这些依赖。

* **Feature 测试:**（有时也被称为 Functional 测试或者 Integration 测试）则对程序的一个特定功能或者一系列交互进行测试。
  * 在 Feature 测试中，通常不会过于关注程序内部的实现细节，而是集中测试整个系统或者程序的一个特定部分是否正常工作。
  * Laravel 中，Feature 测试通常放置在 `tests/Feature` 目录下

## Feature 测试 {id="Feature"}

在一个应用中，我们使用最多的还是 Feature  测试，它的粒度不如单元测试那么小，但是可以覆盖组件间的交互，为应用的质量起到更好的保障。

在大部分的应用中，我们的代码最多的操作是数据库的增删改查。那么针对数据库我们如何测试呢？有两种思路:

* 不测试数据库，数据库的交互使用 Mock。
* 使用 SQLite 数据库，并且切换到内存模式

Mock 我们后文会提到，这里先使用 SQLite 的方式。

### 编写接口 {id="implement-feature"}

在大部分项目中，会进行接口级别的测试，而不是使用单元测试。这不是说单元测试不重要，而是基于测试开发成本和收益的平衡。

首先，我们需要编写一个接口，以创建用户为例:
```PHP
class UserController extends Controller
{
    public function create(): array
    {
        $user = new User();
        $user->username = 'tom';
        $user->password = password_hash('PassW0rd', PASSWORD_DEFAULT);
        $user->save();
        return ['code' => 0, 'message' => 'success'];
    }
}

Route::post('/user', [UserController::class, 'create']);
```
我编写了一个 `UserController::create()` 的接口，接下来针对真个接口进行测试。

### 数据迁移 {id="migration"}

每次运行单元测试之前，Laravel 都会使用 `Migration` 功能来初始化数据库。所以，我们需要先使用它来创建 `users` 数据表:
```Shell
$ php artisan make:migration create_user_table --create=users
```

这条命令会在`database/migrations` 目录下创建一条迁移记录，完整代码如下:
```PHP
return new class extends Migration
{

    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->bigIncrements('id')->comment('主键');
            $table->string('username', 64)->comment('用户名');
            $table->string('password', 128)->comment('密码');
            $table->timestamp('deleted_at')->nullable()->comment('删除时间');
            $table->timestamps();
        });
        // SQLite 不支持 ALTER TABLE 命令添加或者修改表注释，所以跳过
        if (DB::connection()->getPdo()->getAttribute(PDO::ATTR_DRIVER_NAME) !== 'sqlite') {
            DB::statement('ALTER TABLE `users` comment "用户表"');
        }
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```
迁移的时候，会执行 `up` 方法创建表，当回滚的时候，会执行 `down` 方法删除表。初始化表命令如下:
```Shell
$ php artisan migrate
```

### 启用 SQLite {id="using-sqlite"}

首先，我们需要在 `phpunit.xml` 中进行配置，开启 SQLite 数据库以及切换到内存模式（默认是本地文件的模式）。
```xml
<php>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
</php>
```

### 编写测试用例 {id="write-test-case"}

接着我们就可以编写测试用例，并执行测试了，代码如下:
```PHP
class UserControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_create(): void
    {
        $response = $this->postJson('/api/user');
        // 验证请求结果
        $response->assertOk()
            ->assertJsonStructure(['code', 'message'])
            ->assertJson([
                'code' => 0,
                'message' => 'success'
            ]);
        // 验证数据库中的数据是否存在
        $this->assertDatabaseHas('users', [
            'username' => 'tom'
        ]);
    }
}
```

然后在执行如下命令执行测试:
```Shell
php artisan test
```
测试通过的话输出如下:

![test result](http://file-linker.oss-cn-hangzhou.aliyuncs.com/ANoACS7JuiH4XPUQMSPU.png)

### 更多逻辑 {id="more"}

随着业务的变更，我们可能会为创建用户添加更多的逻辑，比如用户名不能重复。代码如下:
```PHP
public function create(): array
{
    $user = User::where('username', 'tom')->first();
    if ($user) {
        throw new Exception('Username already exits');
    }
    // 省略重复的代码
}
```
在原有的逻辑上，我们增加了对用户名是否唯一的判断，如果用户名已经存在，则不允许注册并抛出异常。针对增加的逻辑，我们编写了一个新的测试用例:
```PHP
public function test_is_thrown_if_username_already_exists() {
    User::create([
        'username' => 'tom',
        'password' => password_hash('PassW0rd', PASSWORD_DEFAULT),
    ]);
    $response = $this->postJson('/api/user');
    $response->assertStatus(500);
}
```
在实际的生产中，我们会拦截异常并统一处理返回标准的 JSON 数据，设定错误 code。所以应该在加上对 code 的判断。
