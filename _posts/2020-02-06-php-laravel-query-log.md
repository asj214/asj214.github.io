---
layout: post
title: "라라벨(laravel) 7.x 데이터 베이스 쿼리 로그"
author: "sjahn"
categories: php
tags: [php, laravel]
---

라라벨을 사용하면서 데이터베이스 쿼리로그가 궁금할때가 있다.  
대부분의 프레임워크에서는 기본으로 제공하는 기능들 같지만 라라벨은 아니였다.  
그래서 별로로 추가 시켜주어야한다.  

`app/Providers/AppServiceProvider.php` 에 추가

```php
namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;


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
        //
        // database query logging
        DB::listen(function($query){

            foreach ($query->bindings as $i => $binding){
                if($binding instanceof \DateTime){
                    $query->bindings[$i] = $binding->format('\'Y-m-d H:i:s\'');
                } else {
                    if(is_string($binding)){
                        $query->bindings[$i] = "'$binding'";
                    }
                }
            }

            // Insert bindings into query
            $boundSql = str_replace(['%', '?'], ['%%', '%s'], $query->sql);
            $boundSql = vsprintf($boundSql, $query->bindings);

            Log::info("----------------------------------------------------------------");
            Log::info($boundSql." (".$query->time."s)");

        });
    }
}
```

로그 파일 화면에 출력  
`tail -f storage/logs/laravel.log`
