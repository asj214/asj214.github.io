---
layout: post
title: "라라벨(laravel) 7.x 좋아요 기능 추가"
author: "sjahn"
categories: php
tags: [php, laravel]
---

`Post`에 `Attachment`, `Comment` 이제는 좋아요 기능을 추가할 차례이다.  
좋아요 기능을 작업하면서 내가 좋아요한 대상을 심플하게 구현하고 싶어서 많은 고민을 하였는데, 
아직 완벽한 해결방법을 찾지 못하고 있다.  
추후에 공부를 계속하면서 발견되면 이 포스트를 수정할 예정이다.  

#### 좋아요 마이그레이션 파일 생성  
`artisan make:migration create_likes_table --create=likes`

#### 마이그레이션 파일 작성  

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

#### 데이터베이스 마이그레이션  
`artisan migrate`

#### 모델 생성  
`artisan make:model Like`

#### 모델 파일 작성  

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Like extends Model
{
    //
    protected $table = "likes";

    public function scopeMyLike($query, $type, $id, $user_id){
        return $query->where('like_type', $type)->where('like_id', $id)->where('user_id', $user_id);
    }

    public function scopeMyLikes($query, $type, $user_id, $ids){
        return $query->where('like_type', $type)->whereIn('like_id', $ids)->where('user_id', $user_id);
    }
}
```

좋아요 기능은 별도의 이력이 필요없을 것 같아 softdelete 기능은 사용하지 않았다.
대신에 쿼리스코프 기능을 이용하여, 내가 좋아요를 한 대상을 찾을 수있게 해두었다.

#### `routes/web.php` 에 라우트 추가  

```php
Route::post('posts/{post}/comments', 'PostController@comments')->name('posts.comments');
Route::delete('posts/{id}/comments', 'PostController@comment_destroy')->name('posts.comment_destroy');

Route::post('posts/{post}/like', 'PostController@like')->name('posts.like');
Route::delete('posts/{post}/like', 'PostController@unlike')->name('posts.unlike');

Route::resource('posts', 'PostController');
```

#### `posts@like` 에 기능 추가  

```php
public function like(Request $request, $id){

    $like = new Like();
    $like->user_id = Auth::id();
    $like->like_type = 'posts';
    $like->like_id = $id;
    $like->save();

    $post = Post::find($id);
    $post->like_count = $post->like_count + 1;
    $post->save();

    return redirect()->route('posts.show', ['post' => $id]);

}
```

댓글과 마찬가지로 posts 테이블에 like_count 컬럼을 추가하여, increments, decrements 기능을 구현하였다.

#### `post@unlike` 작성  

```php
public function unlike(Request $request, $id){

    $like_type = $request->input('like_type', 'posts');
    $like = Like::currentUser($like_type, $id, Auth::id())->first();

    $post = Post::find($id);
    $post->like_count = $post->like_count - 1;
    $post->save();

    $like->destroy($like->id);
    return redirect()->route('posts.show', ['post' => $id]);

}
```

추가로 작업하면서 lazy load 를 해야하는 모델들이 점차 늘어나 Post 모델에 자동으로 load 되는 기능을 추가하였다.  

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model {
    //
    use SoftDeletes;

    protected $table = "posts";
    protected $dates = ['deleted_at'];

    protected $with = ['user', 'attachment', 'comments'];


    public function user(){
        return $this->belongsTo('App\User', 'user_id');
    }

    public function attachment(){
        return $this->hasOne('App\Attachment', 'attachment_id')->where('attachment_type', 'posts');
    }

    public function comments(){
        return $this->hasMany('App\Comment', 'commentable_id')->where('commentable_type', 'posts')->orderBy('id', 'desc');
    }

}
```

모델 내부에 `$with` 배열이 바로 그것이다.  
저렇게 해두면 자동으로 관계를 맺고 가지고 오게해준다.

#### `posts@index` 에 like 가져오기  

```php
public function index(Request $request){
    //
    $per_page = $request->input('per_page', 15);

    $posts = Post::orderBy('id', 'desc')->paginate($per_page);
    $likes = Like::myLikes('posts', Auth::id(), $posts->pluck('id'))->get()->pluck('like_id')->toArray();

    return view('posts.list', compact('posts', 'likes'));
}
```

위에서 작성한 `Like` 모델의 스코프를 이곳에서 사용할 차례이다.
일일히 where, whereIn을 사용할수도 있지만 자주 사용할 구석이 보이기도 하고 코드가 길어지는 것 같아서
스코프로 처리하였다.

#### `posts@show` 에 like 가져오기   

```php
public function show($id){
    //
    $post = Post::find($id);
    $post_user_like = Like::myLike('posts', $id, Auth::id())->exists();

    return view('posts.show', compact('post', 'post_user_like'));
}
```

이제 게시판, 첨부파일, 댓글, 좋아요까지 기능을 구현하였다.
다음에는 회원 테이블 쪽을 개선하는 작업을 할 예정이다.