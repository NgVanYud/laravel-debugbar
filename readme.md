## Laravel Debugbar
[![Packagist License](https://poser.pugx.org/barryvdh/laravel-debugbar/license.png)](http://choosealicense.com/licenses/mit/)
[![Latest Stable Version](https://poser.pugx.org/barryvdh/laravel-debugbar/version.png)](https://packagist.org/packages/barryvdh/laravel-debugbar)
[![Total Downloads](https://poser.pugx.org/barryvdh/laravel-debugbar/d/total.png)](https://packagist.org/packages/barryvdh/laravel-debugbar)

### Lưu ý với version 3: Debugbar được enable bởi chính package nhưng vẫn cần phải xét giá trị APP_DEBUG = true
### Đối với những version Laravel trước version 5.5 nên sử dụng [nhánh 2.4](https://github.com/barryvdh/laravel-debugbar/tree/2.4)
Đây là một package dùng để tích hợp  [PHP Debug Bar](http://phpdebugbar.com/) vào trong laravel 5.

+Package này bao gồm 1 ServiceProvider, ServiceProvider này dùng để đăng ký debugbar và đưa dữ liệu ra output. Ta có thể publish các assets và configure package này thông qua Laravel.

+Package này nạp một vài Collectors để làm việc với Laravel và thực hiện 1 couple custom DataCollectors, chỉ dành riêng đối với Laravel.

+Package được cấu hình để hiern thỉ các trang điều hướng và các request Ajax.


![Screenshot](https://cloud.githubusercontent.com/assets/973269/4270452/740c8c8c-3ccb-11e4-8d9a-5a9e64f19351.png)

Lưu ý: Chỉ sử dụng Debugbar trong khi phát triển sản phẩm. Nó có thể làm chậm ứng dụng (vì nó phải thu thập dữ liệu). Khi gặp tình trạng này, hãy thử
vô hiệu hóa một vài Collector.


Package này có bao gồm một vài tùy chọn collectors:

- QueryCollector: hiển thị tất cả các query, bao gồm binding + timing.
- RouteCollector: hiển thị thông tin về Route thời điểm hiện tại.
- ViewCollector: hiển thị các views được load vào thời điểm hiện tại. (Tùy chọn: hiển thị dữ liệu được chia sẻ).
- EventsCollector: hiển thị các event.
- LaravelCollector: hiển thị enviroment và version Laravel (mặc định bị disable).
- SymfonyRequestCollector: thay thế RequestCollector cùng với một vài thông tin thêm về request/response.
- LogsCollector: hiển thị log gần đây nhất từ bộ lưu trữ log (mặc định bị disable).
- FilesCollector: hiển thị các file được require hay include bởi PHP (mặc định bị disable).
- ConfigCollector: hiển thị các giá trị trong file config (mặc định bị disable).
- CacheCollector: hiển thị tất cả các sự kiện cache (mặc định bị disable).

Các collectors dưới đây được khởi động cùng laravel:

- LogCollector: hiển thị tất cả các thông báo.
- SwiftMailCollector và SwiftLogCollector dùng cho Mail.

Các collector mặc định:

- PhpInfoCollector
- MessagesCollector
- TimeDataCollector (Bao gồm thời gian khởi động và chạy ứng dụng)
- MemoryCollector
- ExceptionsCollector


Package này cũng có 1 Facade interface để dễ dàng cho việc ghi các log Message, Exception và Time

## Cài dặt

Bắt buộc phải có composer để sử dụng package này. Và chỉ nên sử dụng package này trong môi trường đang phát triển sản phẩm.

```shell
composer require barryvdh/laravel-debugbar --dev
```

Vì Laravel 5.5 sử  dụng Package Auto-Discovery nên không bắt buộc phải thực hiện thêm ServiceProvider bằng tay. 

Debugbar được kích hoạt khi giá trị của `APP_DEBUG` là `true`.

> Nếu sử dung catch-all/fallback route, cần phải load Debugbar ServiceProvider trước App Service Provider.

### Laravel 5.5+:

Nếu không dùng auto-discovery, thêm ServiceProvider vào trong array provider nằm trong file config/app.php

```php
Barryvdh\Debugbar\ServiceProvider::class,
```

Nếu muốn sử dụng facade để tạo các thông báo log, thêm dòng sau vào facade trong file app.php

```php
'Debugbar' => Barryvdh\Debugbar\Facade::class,
```

Debugbar mặc định sẽ được enable nếu thiết lập giá trị APP_DEBUG=true. Có thể ghi đè giá trị đó trong file config (`debugbar.enable`) hoặc thiết lập cho giá trị của `DEBUGBAR_ENABLE` trong `.env`.

Nếu muốn include hoặc exclude các file vendor (Fontawesome, Highlight.js và jQuery) có thể thiêt lập trong file config. Nếu các vendor này đã được sử dụng rồi thì thiết lập giá trị này là false.

Cũng có thể cài đặt chỉ hiển thị các vendor js hoặc css mà bằng cách thiết lập giá trị trong config là 'js' hoặc 'css'. (Highlight.js sử dụng cả css và js nên cần gán `true`)
 
Sao chép package config vào trong local config với câu lệnh publish:

```shell
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

### Lumen:

Đối với Lumen thì register 1 Provider được thực hiện trong `bootstrap/app.php`:

```php
if (env('APP_DEBUG')) {
 $app->register(Barryvdh\Debugbar\LumenServiceProvider::class);
}
```

Để thay đổi các thiết lập cấu hình, copy file tới folder config và enable nó:

```php
$app->configure('debugbar');
```

## Sử dụng

Có thể thêm các message sử dụng Facade, PSR-3 (debug, info, notice, warning, error, critical, alert, emergency):

```php
Debugbar::info($object);
Debugbar::error('Error!');
Debugbar::warning('Watch out…');
Debugbar::addMessage('Another message', 'mylabel');
```

Thời gian bắt đầu/ kết thúc:

```php
Debugbar::startMeasure('render','Time for rendering');
Debugbar::stopMeasure('render');
Debugbar::addMeasure('now', LARAVEL_START, microtime(true));
Debugbar::measure('My long operation', function() {
    // Do something…
});
```

Hoặc các log exception:

```php
try {
    throw new Exception('foobar');
} catch (Exception $e) {
    Debugbar::addThrowable($e);
}
```

Cũng có một vài helper function sẵn dùng: 

```php
// All arguments will be dumped as a debug message
debug($var1, $someString, $intValue, $object);

start_measure('render','Time for rendering');
stop_measure('render');
add_measure('now', LARAVEL_START, microtime(true));
measure('My long operation', function() {
    // Do something…
});
```
Nếu muốn thêm DataCollectors tự tạo, thông qua Container hoặc Facade

```php
Debugbar::addCollector(new DebugBar\DataCollector\MessagesCollector('my_messages'));
//Or via the App container:
$debugbar = App::make('debugbar');
$debugbar->addCollector(new DebugBar\DataCollector\MessagesCollector('my_messages'));
```

Mặc định, Debugbar chỉ được inject vào trước `</body>`. Nếu muốn tự inject Debugbar, cần phải thiết lập tùy chọn 'inject' thành false và tùy ý sử dụng render

```php
$renderer = Debugbar::getJavascriptRenderer();
```

Lưu ý: Việc không sử dụng auto-inject, sẽ disable thông tin Request, vì điều đó có nghĩa là Debugbar sẽ được inject sau response.
Có thể thêm default_request datacollector vào trong config để thay thế.

## Enabling/Disableing khi run time

Có thể enable hoặc disable debugbar trong khi run time.

```php
\Debugbar::enable();
\Debugbar::disable();
```

Khi đã được enable, các collectors sẽ được thêm vào (và có thể tạo ra những thành phần khác), nên nếu muốn sử dụng debugbar trong production, hãy disable trong file config và chi enable khi cần sử dụng thôi.

## Tích hợp Twig

Laravel Debugbar đi kèm với 2 Twig Extensions. Các extension này đã được kiểm thử với [rcrowe/TwigBridge](https://github.com/rcrowe/TwigBridge) 0.6.x

Thêm các extension dưới đây vào TwigBridge config/extension.php (hoặc register extension bằng tay).

```php
'Barryvdh\Debugbar\Twig\Extension\Debug',
'Barryvdh\Debugbar\Twig\Extension\Dump',
'Barryvdh\Debugbar\Twig\Extension\Stopwatch',
```

Extenion Dump sẽ thay thế [dump function](http://twig.sensiolabs.org/doc/functions/dump.html) thành các variable đầu ra sử dụng DataFormatter. 

Extension Debug thêm 1 hàm `debug()`, hàm này sẽ truyền các variable tới Message Collector thay vì việc hiển thị trực tiếp ra template. Nó dump các tham số, nếu empty thì là tất cả các variable được sử dụng.

```twig
{{ debug() }}
{{ debug(user, categories) }}
```

Extension Stopwatch thêm [stopwatch tag](http://symfony.com/blog/new-in-symfony-2-4-a-stopwatch-tag-for-twig) tương tự vào 1 trong các Symfony/Silex Twigbrigde.

```twig
{% stopwatch "foo" %}
    …some things that gets timed
{% endstopwatch %}
```
