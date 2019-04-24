+++
title = "Laravel 使用 Faker Generator 填充测试数据"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2018-02-05T01:44:58
updated_at = 2018-02-05T01:44:58
description = "简短的描述"
tags = ["php", "laravel"]
+++

## 准备 Model

这个数据填充是基于 Model 的，假设已经有一个保存博客的表叫 posts ，对应的 Model 就是 Post.php

### 创建 Model 和 数据迁移文件

```bash
php artisan make:model Models/Post -m
```

### 配置数据迁移文件

编辑 `database/migrations/2018_02_05_140002_create_posts_table.php`

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->increments('id');
        $table->string('title')->index();
        $table->text('body');
        $table->integer('user_id')->unsigned()->index();
        $table->integer('skill_id')->unsigned()->index();
        $table->integer('comment_count')->unsigned()->default(0);
        $table->integer('view_count')->unsigned()->default(0);
        $table->integer('order')->unsigned()->default(0);
        $table->string('excerpt');
        $table->string('slug')->default('');
        $table->timestamps();
    });
}
```

### 编辑 Model

`app/Models/Post.php`

```php
protected $fillable = [
    'title', 'body', 'user_id', 'skill_id', 'excerpt', 'slug',
];
```

## 创建数据数据工厂

### 创建数据工厂

```bash
php artisan make:factory PostFactory
```

编辑 `database/factories/PostFactory.php`

```php
<?php

use Faker\Generator as Faker;

$factory->define(App\Models\Post::class, function (Faker $faker) {
    $sentence = $faker->sentence();

    // 随机取 5 年前到现在的时间
    $updated_at = $faker->dateTimeBetween('-5 years');
    // 传参为生成最大时间不超过，创建时间永远比更改时间要早
    $created_at = $faker->dateTimeBetween('-5 years',$updated_at);

    return [
        'title' => $sentence,
        'body' => $faker->text(5120),
        'excerpt' => $sentence,
        'created_at' => $created_at,
        'updated_at' => $updated_at,
    ];
});

```

上面示例就是随便生成了一些文章内容，更多的格式见  [fzaninotto/Faker](https://github.com/fzaninotto/Faker#formatters)

## 创建 PostsTableSeeder

```bash
php artisan make:seeder PostsTableSeeder
```

编辑 `database/seeds/PostsTableSeeder.php`

```php
// 创建 100 篇文章
$posts = factory(Post::class)
            ->times(100)
            ->make();

// 写入数据到数据库
Post::insert($posts->toArray());
```

## 执行数据填充

```bash
php artisan db:seed --class=PostsTableSeeder
```

## 后记

### 文章添加额外字段

可以利用 each 函数，在函数中写如额外的内容

如给每一篇文章加上一个用户 ID

```php
$faker = Faker\Factory::create('zh_CN');

$posts = factory(Post::class)
    ->times(2)
    ->make()
    ->each(function ($post) use ($userIds,$faker) {
        $post->user_id  = $faker->randomElement($userIds);
    });
```

### 添加中文支持

可以将 Faker 创建填充数据的时候可以使用中文，不过支持的数据类型比较少，如 用户、地址、手机等。

详细支持的类型 [参考](https://github.com/fzaninotto/Faker/tree/master/src/Faker/Provider/zh_CN)

Laravel 要想实现也比较简单，直接在配置文件中加入 `'faker_locale' => 'zh_CN',` 即可，比如可以加到 `config/app.php` 中

### 参考资料

[数据库测试](https://d.laravel-china.org/docs/5.5/database-testing)

[fzaninotto/Faker](https://github.com/fzaninotto/Faker/)