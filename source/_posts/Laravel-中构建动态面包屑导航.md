---
title: Laravel 中构建动态面包屑导航
tags:
  - PHP
  - 框架
  - Laravel
  - 面包屑
  - 导航
comments: false
img: "/p/building-dynamic-breadcrumbs-in-laravel/head.png"
abbrlink: building-dynamic-breadcrumbs-in-laravel
date: 2019-08-01 16:37:40
---

## 使用面包屑导航的重要性

在多年前，WEB 开发者大多关心网站的逻辑功能(后端)和页面对用户的可传达信息(前端)，并没有太多关心前端页面的样式呈现。当然，造成这个的原因方方面面，当前没有合适的开发技术去造成复杂的网页，为数不多的可以使用的技术却非常麻烦。

今天，情况大变。UI、UX 在 WEB 阶断所占的地位跟后端的逻辑功能一样重要。现在有很多实用的工具可以使我们的应用程序对用户更加易用。但是也有一些问题需要小心处理。

用户并不喜欢对话框主动出现在他们脸前，当然这并不是让用户感到抓狂的唯一交互。当前用户不知道自己浏览的网页内容在什么层次、目录和分类下更是难受至极。

那么"面包屑导航(breadcrumbs)"就可以简单地将网页位置定位问题解决。

> 面包屑导航提供了一套导航系统去帮助用户定位当前自己浏览网页的位置和对其它网页的关联性。"面包屑"的名称来源于一个童话故事《韩塞尔与葛雷特》，网页中对用户使用了同样原理的寻路方式。网站提供面包屑导航可以减少用户操作，让用户可以更快的找到需要的信息。

在 Laravel 框架中设置面包屑导航功能相当简单。一个 `composer` 扩展包即可实现大部分的功能逻辑。下面我们将会看一下如何使用这个扩展包。

你可以在 [这里](https://github.com/Jordanirabor/Laravel-Breadcrumbs)下载文章中提到的所有代码

## 初始化一个新的 Laravel 程序

```bash
laravel new breadcrumbs
```

OR

```bash
composer create-project --prefer-dist laravel/laravel breadcrumbs
```

初始化完成后我们安装面包屑导航扩展 [Github](https://github.com/davejamesmiller/laravel-breadcrumbs)

```
composer require davejamesmiller/laravel-breadcrumbs
```

## 创建一个请求路由

我们在 `routes/web.php` 文件中创建一些请求路由。 这些路由将会在后面的功能介绍中被使用到。

```php
Route::get('/',  ['as' => 'home', 'uses' => 'MainController@home']);

Route::get('/continent/{name}',  ['as' => 'continent', 'uses' => 'MainController@continent']);

Route::get('/country/{name}',  ['as' => 'country', 'uses' => 'MainController@country']);

Route::get('/city/{name}',  ['as' => 'city', 'uses' => 'MainController@city']);
```

为了演示此扩展的强大性，我们为应用程序创建三个 `Model` ： `Continent`, `Country`, `City`。 其中 `Continent` 对 `Country` 为一对多关联关系，`Country` 对 `City` 也为一对多关联关系。

在命令中输入下面的命令进行创建：

```bash
php artisan make:model Continent
php artisan make:model Country
php artisan make:model City
```

同样的我们创建三个数据库表 `migration` 文件

```bash
php artisan make:migration continents
php artisan make:migration countries
php artisan make:migration cities
```

下面我们在三个 `Model` 文件中定位关联关系。

_app/Continent.php_

```php
namespace App;
use App\Country;

use Illuminate\Database\Eloquent\Model;

class Continent extends Model
{
    public function country(){
        return $this->hasMany(Country::class);
    }
}
```

_app/Country.php_

```php
namespace App;

use App\City;
use App\Continent;

use Illuminate\Database\Eloquent\Model;

class Country extends Model
{
    protected $guarded = [];
    public function city(){
        return $this->hasMany(City::class);
    }

    public function continent(){
        return $this->belongsTo(Continent::class);
    }
}
```

_app/City.php_

```php
namespace App;

use App\Country;

use Illuminate\Database\Eloquent\Model;

class City extends Model
{
    protected $guarded = [];
    public function country(){
        return $this->belongsTo(Country::class);
    }
}
```

> 注意：我们这里把 `guarded` 属性设置为空，是因为我们会做大量的 DB 操作，为了省时间，关闭验证。在生产环境中不可以这么做！

现在，我们创建一个 `Controller` 来处理请求。并定义了一些 `acttion` 来返回 `View` 渲染。

输入下面的命令进行 `Controller` 的创建：

```
php artisan make:controller MainController
```

填充 `action` 定义

_app/Http/Controllers/MainControllers.php_

```php
namespace App\Http\Controllers;
use App\Continent;
use App\Country;
use App\City;
use Illuminate\Http\Request;

class MainController extends Controller
{
    public function home(){
        return view('home');
    }

     public function continent($name){
           $continent = Continent::where('name', $name)->first();
        return view('continent', compact('continent'));
    }

     public function country($name){
           $country = Country::where('name', $name)->first();
        return view('country', compact('country'));
    }

     public function city($name){
           $city = City::where('name', $name)->first();
        return view('city', compact('city'));
    }
}
```

我们在 `Model` 中定义的关联关系，可以被扩展包自动探测到。下面，我们创建一些 `View` 文件，来跟 `Model` 文件进行一对应。在目录 `resources/views` 中创建下列文件。

```
home.blade.php
continent.blade.php
country.blade.php
city.blade.php
```

文件创建完成后在这些 `View` 文件中分别录入下面的代码。

_resources/views/home.blade.php_

```php
<!DOCTYPE html>
<html>
<head>

<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">

 </head>
 <body>

{{ Breadcrumbs::render('home') }}

</body>
</html>
```

_resources/views/continent.blade.php_

```php
<!DOCTYPE html>
<html>
<head>

<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">

 </head>
 <body>

{{ Breadcrumbs::render('continent', $continent) }}

</body>
</html>
```

_resources/views/country.blade.php_

```php
<!DOCTYPE html>
<html>
<head>

<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">

 </head>
 <body>

{{ Breadcrumbs::render('country', $country->continent, $country) }}

</body>
</html>
```

_resources/views/city.blade.php_

```php
<!DOCTYPE html>
<html>
<head>

<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">

 </head>
 <body>

{{ Breadcrumbs::render('city', $city->country->continent, $city->country, $city) }}

</body>
</html>
```

`Continent` 和 `Country` 模式是 `一对多` 的关联关系。一个`Continent` 会有多个 `Country` ， 一个 `Country` 会有多个 `City`。

最后我们来完善我们的 `migration` 数据文件。

_database/migrations/2017_11_02_092826_continents.php_

```php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class Continents extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('continents', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```

_database/migrations/2017_11_02_092835_countries.php_

```php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class Countries extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('countries', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('continent_id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```

_database/migrations/2017_11_02_092845_cities.php_

```php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class Cities extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('cities', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('country_id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```

我们运行 `migrate` 命令来实现真实数据库表的创建：

```
php artisan migrate
```

我们通过`Seeder` `Factory` 技术来进行数据的指插入。

运行下面的命令

```
php artisan make:factory ContinentFactory --model=Continent
php artisan make:factory CountryFactory --model=Country
php artisan make:factory CityFactory --model=City
```

上面的命令会创建三个文件。

_database/factories/ContinentFactory.php_

```php
use Faker\Generator as Faker;

$factory->define(App\Continent::class, function (Faker $faker) {
    return [
        //
    ];
});
```

_database/factories/CountryFactory.php_

```php
use Faker\Generator as Faker;

$factory->define(App\Country::class, function (Faker $faker) {
    return [
        //
    ];
});
```

_database/factories/CityFactory.php_

```php
use Faker\Generator as Faker;

$factory->define(App\City::class, function (Faker $faker) {
    return [
        //
    ];
});
```

修改 `DatabaseSeeder.php` 文件来完成数据的指插入业务 :

```php
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {

      factory(App\City::class)->create([
        'name' => 'Johannesburg',
        'country_id' => function(){

            return factory(App\Country::class)->create([
                'name' => 'South Africa',
                'continent_id' => function(){

                    return factory(App\Continent::class)->create([
                        'name' => 'Africa' ])->id;
                }
                ])->id;
        }
        ]);
    }
}
```

运行此命令将数据插入表中。

```bash
php artisan db:seed --class=DatabaseSeeder
```

## 初始化面包屑配置文件

我们在路由文件夹内创建一个 `breadcrumbs.php` 文件。这个文件会在一个面包屑导航需要渲染时引用。

第一个方法定义了我们网站程序的 `Home` 导航，会在路由 `route('home')` 访问时被使用。

_routes/breadcrumbs.php_

```php
Breadcrumbs::register('home', function ($breadcrumbs) {
     $breadcrumbs->push('Home', route('home'));
});
```

## 渲染一个静态面包屑导航

在上面创建的导航方法中，我们发现 `Breadcrumbs` 类通过调用一个静态方法 `register` 注册了一个新的 `Home` 面包屑导航。注册逻辑的闭包中传入的 `$breadcrumbs` 参数 push 了一个 `Home` 导航的链接地址。

```php
$breadcrumbs->push('Home', route('home'));
```

这个 `Home` 导航是硬编码到了逻辑中，会在面包屑需要渲染的时候或在其它路由需要 `Home` 做为一个导航路径链的时候起作用。

面包屑的渲染方式：

_resources/views/home.blade.php_

```php
{{ Breadcrumbs::render('home') }}
```

渲染时需要接收一个面包屑导航名称的参数 ，在更加复杂的导航链中，`render` 方法同样接受传入 `Model` 关系对象做为参数。

## 渲染动态面包屑

大多数情况下我们的网站数据是动态变化的，同样的面包屑导航的数据也需要根据当前场景的不同完成动态生成与切换。

![img](1.png)

同样的，为了实现此功能，我们在面包屑定义文件中加下面的代码

_routes/breadcrumbs.php_

```php
Breadcrumbs::register('continent', function ($breadcrumbs, $continent) {
    $breadcrumbs->parent('home');
    $breadcrumbs->push($continent->name, route('continent', ['name' => $continent->name]));
});
```

以上代码中，我们注册了一个 `Continent` 导航元素。同时闭包内接收了一个 `$continent` 的 `model` 实例，这个实例提供了当前元素需要渲染的名称。

同样的，我们添加导航的渲染逻辑代码，这里会将 `$continent` 实例带入，来传递进上面定义的闭包中去。

_resources/views/continent.blade.php_

```php
{{ Breadcrumbs::render('continent', $continent) }}
```

下面方法下网站页面，就可以看下下面的效果了。

![img](2.png)

## 渲染一个完整的面包屑导航

我们在上面完成了渲染一个单独的面包屑元素。现在，我们来渲染一个完整的面包屑导航。根据本文开关提到的数据模型，我们会渲染一个城市导航。

下面是我们定义的导航的所有代码

_routes/breadcrumbs.php_

```php
Breadcrumbs::register('home', function ($breadcrumbs) {
     $breadcrumbs->push('Home', route('home'));
});

Breadcrumbs::register('continent', function ($breadcrumbs, $continent) {
    $breadcrumbs->parent('home');
    $breadcrumbs->push($continent->name, route('continent', ['name' => $continent->name]));
});

Breadcrumbs::register('country', function ($breadcrumbs, $continent, $country) {
    $breadcrumbs->parent('continent', $continent);
    $breadcrumbs->push($country->name, route('country', ['name' => $country->name]));
});

Breadcrumbs::register('city', function ($breadcrumbs, $continent, $country, $city) {
    $breadcrumbs->parent('country', $continent, $country);
    $breadcrumbs->push($city->name, route('city', ['name' => $city->name]));
});
```

我们添加了两个面包屑导航元素 `Country` 和 `City` 用来将 一个城市所属的国家，一个国家所在的洲这个面包屑导航链完整的展示出来。

在每个导航定义的闭包内 `parent` 方法定义了当前导航的父级导航是什么。

下面就是调用代码。

_resources/views/city.blade.php_

```php
{{ Breadcrumbs::render('city', $city->country->continent, $city->country, $city) }}
```

这样我们就完成了一个完整的面包屑导航.

## 写在最后

上面的实例只需要几行代码就可以完整展示一个导航，为网站用户体验提升一个新的档次。为用户创建一个容易使用和识别的高人性化交互体验服务。
