---
layout: post
title: "라라벨(laravel) 7.x 좋아요 기능 추가"
author: "sjahn"
categories: php
tags: [php, laravel]
---

`artisan make:migration create_likes_table --create=likes`

```php
public function up()
{
    Schema::create('likes', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id');
        $table->string('like_type');
        $table->integer('like_id');
        $table->timestamps();
        $table->index(['like_type', 'like_id']);
    });
}
```

`artisan migrate`

`artisan make:model Like`