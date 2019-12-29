本教程主要针对为laravel 6 配置reCAPTCHA v2.0 checkbox的情况

# 内容目录：
### 第一部分： 什么是reCAPTCHA？为何要配置它？如何选择适合自己网站的身份验证方式？
##### 1.1 reCAPTCHA是什么？
##### 1.2 reCAPTCHA的优缺点
##### 1.3 laravel验证码平行比较：mews/captcha？为何选择reCAPTCHA？
##### 1.4 reCAPTCHA的各个版本介绍，及如何选择自己适用的reCAPTCHA：
##### 1.5 为何选择自行配置reCAPTCHA，而不是使用目前已有的、已经整合在laravel系统的包？
### 第二部分：为laravel6配置reCAPTCHA v2.0 checkbox 的手把手教程
##### 2.1 安装含有auth的全新laravel 6程序
##### 2.2 配置reCAPTCHA
##### 2.3 bonus：如何兼容中文
***
前日紧急为个人站点配置了google reCAPTCHA验证系统，避免盗号攻击。将经验总结写作下列教程，回馈社区。

# 第一部分： 什么是reCAPTCHA？为何要配置它？如何选择适合自己网站的身份验证方式？

## 1.1 reCAPTCHA是什么？
**reCAPTCHA** 是 **CAPTCHA** 的衍生词。CAPTCHA是 Completely Automated Public Turing test to tell Computers and Humans Apart 的缩写，意为“全自动区分计算机和人类的图灵测试”。reCAPTCHA是谷歌基于这个思路开发的一套具体的验证规则。目前，它具有多个版本。


## 1.2 reCAPTCHA的优缺点
### 优点：
- 独立的软件包，安装和“拆卸”都便利
- 联网谷歌，根据往期记录决定用户的“真实性”，相对难以破解，更安全。
- 免费
### 缺点：
- 联网谷歌，你懂的……联网问题。
- 语言需要额外配置。
- 一部分验证的思维方式，不太符合中文生活经验与使用习惯。


## 1.3 laravel验证码平行比较：reCAPTCHA VS mews/captcha
提到```reCAPTCHA```，不得不先介绍laravel的另一个常用的验证码系统 ```mews/captcha```。

```mews/captcha``` 是集合在laravel系统上的一个**方便好用美观**的图片验证码系统。它随机生成一个含有邀请码字符的图片，要求用户进行识别，输入图上的字符，完成验证。官方代码地址：https://github.com/mewebstudio/captcha

这个系统文档完备，非常方便配置，而且对于流量不高的普通个人工程来说，具有不错的防护效果。```mews/captcha``` 本身的安装文档就很完整，可以直接照着安装。如果你想要中文的安装介绍，可以参考这个网址：https://learnku.com/laravel/t/2895/extension-recommended-mewscaptcha-image-authentication-code-solution

我研究过很多目前已有的、免费可用的、兼容laravel的验证码系统。坦诚地说，**_如果你是刚开始进行建站的开发者，使用laravel，又需要配置验证码，```mews/captcha``` 是你的首选_**。尽量主要精力花在你的核心服务逻辑上。

但是……我们实际使用 ```mews/captcha``` 的悲惨结果是，它**比较容易被破解**。基于本站实际体验，后台数据跟踪发现，在使用它的第一个月，默认版式的验证码被破解。更换到inverse图片之后的大概第二个月，“inverse”版式也被破解了。

#### 因此，紧急更换并配置reCAPTCHA，成了我们被迫的、无奈的结果。


***
## 1.4 reCAPTCHA的各个版本介绍，及如何选择自己适用的reCAPTCHA：
######  reCAPTCHA v1.0：已废弃
###### reCAPTCHA v2.0: 目前主流使用的reCAPTCHA系统，具有下面几个“变种”：
- **“im not a bot checkbox”** 通过点击方框，决定用户是否真人。如果用户的“可疑性”比较高，进一步要求用户做图片识别（比如找公交车、找人行横道……）直到用户作出正确的判断再放行。
 - **“invisible”recaptcha** 隐藏在幕后，直接通过用户的行为判断是否真人的recaptcha，是reCAPTCHA 3.0 的前置产品。个人理解，目前的v3.0基本完全覆盖了这部分的功能，因此没有采用的必要。
 - 专门适用android的recaptcha
###### reCAPTCHA v3.0：最新推出的reCAPTCHA
只对网站提供用户的安全分值（0-1，1代表真人，0代表bot），由网站自行判断下一步要做什么。
### 结论1: 对于个人非安卓的开发者，只需要从reCAPTCHA v2.0 checkbox和reCAPTCHA v3.0中挑一个就可以了。


以下继续分点阐述这两个版本recaptcha的好坏比较，方便用户选择最适合自己的验证方式：

#### reCAPTCHA v2.0 checkbox
###### 主要好处：
有预置的fallback。如果用户的分数较为可疑，并不会拒绝用户，而是要求用户用进一步的图片识别继续验证自己身份，验证之后就允许访问。
###### 主要坏处：
图片识别过程非常繁琐，可能会有较大的用户抱怨
#### reCAPTCHA v3.0
###### 主要好处：
- 可以用在你网站的所有页面，并根据不同的页面配置不同的处理决定
- 可以根据实际情况，自定义对分数的后续处理（比如说，在某些网站，0.5是可疑用户不能允许访问，而在另一些网站，没必要阻止0.5的用户访问）
###### 主要坏处：
- 如果你因为分数很低直接拒绝了某些用户，你可能会永久地失去他们。因此，recaptcha3.0需要自行配置合理的fallback办法，这对个人开发者来讲是一个比较大的业务负担。

### 结论2: **目前reCAPTCHA 3.0并不能直接取代reCAPTCHA 2.0 checkbox**（这可能也是为什么谷歌并没有停止2.0业务的原因）。**开发者应根据自己的情况自主选择，而不是片面升级到最新版本的reCAPTCHA**。

##### 结合个人当前的主要需求：在注册、登陆等重要页面进行身份验证，避免bot攻击，但也不要把真人踢走的需求，我们选择reCAPTCHA v2.0 checkbox作为新的验证手段。
***

# 1.5 为何选择自行配置reCAPTCHA，而不是使用目前已有的、已经整合在laravel系统的包？
主要是因为，大部分laravel和reCAPTCHA系统的整合包，并不支持reCAPTCHA的global配置（也就是说会被墙），需要自己进行微小的调整。其次，因为命名的问题，他人的reCAPTCHA整合包，可能会和其他的验证码方式冲突，如果开发者打算在系统里加入多个验证方式，需要特别注意这一点。
实际上，个人自行配置reCAPTCHA非常简单，也很灵活，命名冲突也非常好解决。

# 2 个人自配置reCAPTCHA教程：

接下来的教程里，将新建一个全新空白laravel工程，并为它配置reCAPTCHA v2.0 checkbox系统
全部代码见：https://github.com/lyn510/reCAPTCHA
我们这就开始吧！
## 2.1 安装含有auth的全新laravel6程序
首先安装一个版本为 laravel 6 的全新程序。 这和laravel5的配置过程会略有不同。
```
$ composer create-project laravel/laravel reCAPTCHA
```
接下来安装默认的auth身份验证方式（就是基础的登陆注册系统）
```
$ composer require laravel/ui --dev
```
上面的指令安装了默认ui
```
$ php artisan ui vue --auth
```
上面的指令安装了登陆界面
```
$ npm install && npm run dev
```
上面指令会compile对应的css文件，让ui界面的内容得以被呈现出来。它会需要比较长的时间来运行，视网络环境而定，请耐心等待。

接下来，为了正常使用登陆系统，需要建立对应的数据库。本教程里使用的是mysql。

建立一个mysql数据库，数据库的名字叫```reCAPTCHA-auth```, 用户名```root```, 密码留空。

配置```.env```文件，使程序能连接到数据库：
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=reCAPTCHA-auth
DB_USERNAME=root
DB_PASSWORD=
```

之后，回到terminal，运行migrate命令
```
$ php artisan migrate
```
接着查看一下目前的情况
```
$ php artisan serve
```
从浏览器打开对应的链接```http://127.0.0.1:8000```，就可以正常的登陆系统。（在terminal中，运行 control+C，就可以退出）。

## 2.2 配置reCAPTCHA
下面，我们将reCAPTCHA安排进来。
#### 2.2.1 首先使用composer，安装官方包：
```
$ composer require google/recaptcha
```
接着一步步完成后端的配置。
#### 2.2.2 建立一个extended validator来进行对应的测试。
新建文件：```app\Validators\reCaptchaCheckbox.php```
```
<?php
namespace App\Validators;

use ReCaptcha\ReCaptcha;

class reCaptchaCheckbox{

    public function validate($attribute, $value){
        $captcha = new ReCaptcha(env('RECAPTCHA_CHECKBOX_SECRET'));
        $response = $captcha->verify($value, $_SERVER['REMOTE_ADDR']);
        return $response->isSuccess();
    }

    public function message($message, $attribute, $rule, $parameters){
        return ;
    }

}
```
下面将validator注册进入默认的validation系统
修改```app\Providers\AppServiceProvider.php```
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        // 增加这行代码，扩展默认 Validator
        Validator::extend('recaptcha', 'App\Validators\reCaptchaCheckbox@validate');
    }
}
```
上面这里要增加两行代码，一行调度validator，另一行扩展validator的boot方式。


#### 2.2.3 接着我们要求，在用户登录（login）的时候，需要首先进行人机验证。
修改文件```app\Http\Controllers\Auth\LoginController.php```
```
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Providers\RouteServiceProvider;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Http\Request;

class LoginController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles authenticating users for the application and
    | redirecting them to your home screen. The controller uses a trait
    | to conveniently provide its functionality to your applications.
    |
    */

    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = RouteServiceProvider::HOME;

    // 这里，重写login方式，使得recaptcha的验证成为这部分必须的验证环节
    public function login(Request $request)
    {
       $request->validate([
            'g-recaptcha-response' => 'required|recaptcha'
        ]);
        $this->validateLogin($request);

        if ($this->hasTooManyLoginAttempts($request)) {
            $this->fireLockoutEvent($request);
            return $this->sendLockoutResponse($request);
        }

        if ($this->attemptLogin($request)) {
            return $this->sendLoginResponse($request);
        }

        $this->incrementLoginAttempts($request);

        return $this->sendFailedLoginResponse($request);
    }

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }
}
```

#### 2.2.4 再下面，我们要求前端展示这个人机验证的视图
修改文件```resources\views\auth\login.blade.php```
```
...
<div class="g-recaptcha" data-sitekey="{{ env('RECAPTCHA_CHECKBOX_SITEKEY') }}"></div>
<button type="submit" class="btn btn-primary">
                                    {{ __('Login') }}
</button>
```
修改文件```resources\views\layouts.app.blade.php```
 ```
...
<!-- Scripts -->
    <script src="{{ asset('js/app.js') }}" defer></script>
    <script src="https://www.recaptcha.net/recaptcha/api.js"></script>
...
```
注意上面这里，使用```www.recaptcha.net```，而非```www.google.com```，来避免在大陆的访问被墙。

#### 2.2.5 最后别忘了配置开发者专用的key
首先登陆Recaptcha注册官网：https://www.google.com/recaptcha/admin#list
选择v2 checkbox，输入对自己网站的配置即可
![reCAPTCHA申请](https://upload-images.jianshu.io/upload_images/12903084-63ce296773ef1ce2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此处domain选填了127.0.0.1，因为有本地测试情况。

确认后，就会获得相关key
![从google申请获取的开发者使用reCAPTCHA的相关key](https://upload-images.jianshu.io/upload_images/12903084-e6409c7216594d87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


配置到```.env```里：
```
RECAPTCHA_CHECKBOX_SECRET=6Lc6u8oUAAAAAPI4a0lYImLIWyR50xxxxxxxx
RECAPTCHA_CHECKBOX_SITEKEY=6Lc6u8oUAAAAALvO2Tfjh5zRheJGuyxxxxxxx
```
![注册页面-人机验证-中文](https://upload-images.jianshu.io/upload_images/12903084-09a14d6a4ebdce42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所有程序端的配置就完成了
别忘了重新配置目录
```
$ composer dump-autoload
```
重新打开页面
```
$ php artisan serve
```
![注册页面包含reCAPTCHA人机验证办法](https://upload-images.jianshu.io/upload_images/12903084-b22d472317cd9686.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时，只要通过人机测试，就可以正确登陆。

## 2.3 bonus：如何兼容中文
#### 2.3.1 如何配置中文reCAPTCHA界面？
###### 2.3.1.1 方法1: 可以直接要求js配置为中文
修改文件```resources\views\layouts.app.blade.php```
```
...
<!-- Scripts -->
    <script src="{{ asset('js/app.js') }}" defer></script>
    <script src="https://www.recaptcha.net/recaptcha/api.js?hl=zh-CN"></script>
...
```

###### 2.3.1.2 方法2: 也可以设置要求js响应系统环境语言。
修改文件：```resources\views\layouts.app.blade.php```
```
...
<!-- Scripts -->
    <script src="{{ asset('js/app.js') }}" defer></script>
    <script src="https://www.recaptcha.net/recaptcha/api.js?hl={{ app()->getLocale() }}"></script>
...
```
注意这里app()->getLocale()需要自己按照官方doc继续配置多语言。

#### 2.3.2 如何配置中文reCAPTCHA界面？
很简单，修改文件```resources/lang/zh-CN/validation.php```
```
...
'custom'               => [
        'g-recaptcha-response' => [
            'required' => '请先通过人机身份验证，再提交信息。如果相关测试不能正确显示，请检查浏览器设置。',
        ]
    ],
...
```


## About This tutorial
This is a open-sourced tutorial about integrating laravel framework with google reCAPTCHA 2.0 checkbox.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
