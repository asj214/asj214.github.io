---
layout: post
title: "라라벨(laravel) 7.x 시작하기 (3)"
author: "sjahn"
categories: php
tags: [php, laravel]
image: laravel_logo.jpg
---

<br>
### 라라벨(laravel) 7.x 시작하기 (3)
<br>
[이전글](/php/php-laravel-install.html)에서는 `composer`로 라라벨을 설치하고, 
laravel/ui 라는 패키지를 설치하여 회원가입, 로그인 기능 구현을 하였다.  
<br>
이번에는 회원 테이블에 최종 로그인 날짜를 기록하는 기능을 추가하고, 간단한 게시판을 만들어볼 예정이다.  
<br>
기존에 라라벨, 장고, 루비온더레일즈같은 프레임워크를 사용해본 경험이 없는 사람들에게는 많이 생소한 개발방식이라 이 방식을 익히는 것이 굉장히 중요하다.  
<br>
가장 핵심이 되는 것은 `artisan`을 이용한 db 마이그레이션, 모델 컨트롤러 생성이다.  
<br>

#### 최종 로그인 컬럼 추가하기
1. 마이그레이션 파일 생성  
    `artisan make:migration add_columns_in_users --table=users`
2. `database/migrations`에 새로 생성된 마이그레이션 파일에 컬럼 추가 코드 작성
    ```php
    public function up(){

        Schema::table('users', function (Blueprint $table){
            // 최근 접속 일
            $table->datetime('last_login_at')->nullable()->after('remember_token');
        });

    }
    ```
3. 마이그레이션 실행  
    `artisan migrate`
4. `app/Http/Controllers/Auth/LoginController.php` 에 메소드 추가
    ```php
    public function authenticated(Request $request, $user){
        $user->update(['last_login_at' => date('Y-m-d H:i:s')]);
    }
    ```
5. `app/User.php` 에 `$fillable`에 `last_login_at` 추가
    ```php
    protected $fillable = [
        'name', 'email', 'password', 'last_login_at'
    ];
    ```

이제 로그인을 할때 마다 `users.last_login_at` 에 날짜가 갱신된다.
<br>
<br>

#### 게시판 만들기

1. 마이그레이션 파일 생성
    `artisan make:migration create_posts_table --create=posts`

2. 마이그레이션 파일 작성
    ```php
    public function up(){
        Schema::create('boards', function (Blueprint $table){
            $table->bigIncrements('id');
            $table->integer('user_id'); // 작성자 아이디
            $table->string('title'); // 제목
            $table->text('body'); // 본문
            $table->timestamps(); // 생성일, 수정일
            $table->softDeletes(); // 삭제일
            $table->index('deleted_at');
        });
    }
    ```

3. 마이그레이션
    `artisan migrate`

4. 모델 생성
    `artisan make:model Post`

5. 컨트롤러 생성
    `artisan make:controller PostController --resource`
    > --resource 를 추가시키면 컨트롤러에 기본적인 CRUD 메소드들을 알아서 생성해준다.

6. `routes/web.php`에 라우트 추가
    ```php
    Route::resource('posts', 'PostController');
    ```
    이렇게 간단하게 추가가 가능하지만... 실제 업무에서 쓸때는 개별로 추가하는 방식을 사용하는 사례가 많다.

7. 라우트 확인
    `artisan route:list --name=posts`

8. `resources/views/layouts/app.blade.php` gnb에 post 링크 추가
    ```php
    <ul class="navbar-nav mr-auto">
        <li class="nav-item dropdown">
            <a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                {% raw %}{{ __('Post') }}{% endraw %} <span class="caret"></span>
            </a>
            <div class="dropdown-menu dropdown-menu-right" aria-labelledby="navbarDropdown">
                <a class="dropdown-item" href="{% raw %}{{ route('posts.index') }}{% endraw %}">{% raw %}{{ __('List') }}{% endraw %}</a>
                @if(Auth::guard()->check())
                <a class="dropdown-item" href="{% raw %}{{ route('posts.create') }}{% endraw %}">{% raw %}{{ __('Write') }}{% endraw %}</a>
                @endif
            </div>
        </li>
    </ul>
    ```
    중간에 `@if(Auth::guard()->check())`라는 조건을 걸어 로그인한 사용자한테만 글쓰기 버튼은 노출되는 것으로 처리

9. `app/Http/Controllers/PostController.php` 에 사용자 인증 조건 추가
    ```php
    public function __construct(){
        // 사용자 권한
        $this->middleware('auth', ['except' => ['index', 'show']]);
    }
    ```