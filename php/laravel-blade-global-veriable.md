+++
title = "Laravel 配置 Blade 全局变量"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2017-09-18T02:08:46
updated_at = 2017-09-18T02:08:46
description = "Laravel 配置 Provider，支持全局的模板变量"
tags = ["php"]
+++

这个配置的作用就是配置一个每一个页面都可以访问的变量，比如菜单，如果菜单是动态生成的,
每一个页面都加载一个会非常的不方便，代码冗余量也是非常大的，所以这个配置是非常有必要的。

## 配置 Provider

这个 `Provider` 可以直接使用 `AppServiceProvider`，也可以自己写一个，
并在 `config/app.php` 中的 `providers` 数组中添加，进行注册。

## Provider 的 boot 方法中的实现

假设我们需要注册的菜单内容在 `layouts.nav` 模板中，我们的 boot 方法中写入下面内容

```php
view()->composer('layouts.nav', function ($view) {
    // 这里的菜单是随意写的，可以根据实际情况去获取，比如从数据库中
    $menus = ['主页','文章'];
    $view->with('menus',$menus);
});
```

通过上面的赋值后，模板 layouts.nav 就可以在任何页面中使用 $menus 变量了，
就不需要每一个控制器都去 with()