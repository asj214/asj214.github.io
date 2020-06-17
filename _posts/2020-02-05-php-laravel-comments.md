---
layout: post
title: "라라벨(laravel) 7.x 댓글 기능 추가"
author: "sjahn"
categories: php
tags: [php, laravel]
---


comments 마이그레이션 파일 생성  

`artisan make:migration create_comments_table --create=comments`

마이그레이션 코드 작성  
```php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id');
        $table->string('commentable_type');
        $table->integer('commentable_id');
        $table->text('body');
        $table->timestamps();
        $table->softDeletes();
        $table->index(['deleted_at', 'commentable_type', 'commentable_id']);
    });
}
```

마이그레이션  
`artisan migrate`

모델 생성  
`artisan make:model Comment`

모델 작성  
```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Comment extends Model
{
    //
    use SoftDeletes;

    protected $table = "comments";
    protected $dates = ['deleted_at'];
}
```

`Post` 모델에 `Comment` 관계 설정  
```php
class Post extends Model {
    //
    use SoftDeletes;

    protected $table = "posts";
    protected $dates = ['deleted_at'];

    public function attachment(){
        return $this->hasOne('App\Attachment', 'attachment_id')->where('attachment_type', 'posts');
    }

    public function comments(){
        return $this->hasMany('App\Comment', 'commentable_id')->where('commentable_type', 'posts');
    }

}
```

`Post`와 `Comment`는 1:N의 관계로 설정하므로 `hasMany`의 관계로 정하였다.  

`routes.php`에 코멘트 라우트 추가  
```php
Route::post('posts/{post}/comments', 'PostController@comments')->name('posts.comments');
Route::delete('posts/{id}/comments', 'PostController@comment_destroy')->name('posts.comment_destroy');
Route::resource('posts', 'PostController');
```

이전에 `posts` 라우트를 추가한 곳의 위에다 추가해야한다.  

라우트 추가 확인  
`artisan route:list --name=posts`

`show.blade.php` 에 `comment` form 추가  

```html
<div class="card-body">
    <form method="post" action="{% raw %}{{ route('posts.comments', ['post' => $post->id]) }}{% endraw %}">
        @csrf
        <div class="form-group">
            <label for="body">댓글</label>
            <textarea name="body" id="body" class="form-control" aria-label="With textarea" required></textarea>
        </div>
        <input type="submit" class="btn btn-block btn-light btn-outline-secondary" value="작성" />
    </form>
</div>
```

`Post` 모델에 `comment_count` 컬럼 추가  

`artisan make:migration add_column_post_table --table=posts`
```php
public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        //
        $table->integer('comment_count')->default(0)->after('body');
    });
}

public function down()
{
    Schema::table('posts', function (Blueprint $table) {
        //
        $table->dropColumn('comment_count');
    });
}
```

`artisan migrate`


`PostController.php` 에 `comments` 기능 추가  

```php
public function comments(Request $request, $id){

    $comment = new Comment;
    $comment->user_id = Auth::id();
    $comment->commentable_id = $id;
    $comment->commentable_type = 'posts';
    $comment->body = $request->body;
    $comment->save();

    $post = Post::find($id);
    $post->comment_count = $post->comment_count + 1;
    $post->save();

    return redirect()->route('posts.show', ['post' => $id]);

}
```

`comment` 작성 후 `post`에 `comment_count` 를 증가시키는 기능을 추가하였다.  

`PostController.php` 에 `comment_destroy` 기능 추가  

```php
public function comment_destroy(Request $request, $id){

    $comment = Comment::find($id);

    $post = Post::find($comment->commentable_id);
    $post->comment_count = $post->comment_count - 1;
    $post->save();

    $comment->destroy($id);

    return redirect()->route('posts.show', ['post' => $post->id]);

}
```
