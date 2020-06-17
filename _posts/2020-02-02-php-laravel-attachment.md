---
layout: post
title: "라라벨(laravel) 7.x 첨부파일 기능 추가"
author: "sjahn"
categories: php
tags: [php, laravel]
---

<br>
[이전글](/php/php-laravel-post.html)에서 작업한 Post에 첨부파일 업로드 기능을 추가하는 것을 기록한다.  
해당 `attachments` 테이블은 추후에 예정된 어드민 배너관리자에서도 쓰이게할 예정이고, 첨부파일을 이용하는 모든 기능에서 쓰일 예정이라
범용성이 있도록 설계를 하였다.
<br>

#### 첨부파일 테이블 생성

`artisan make:migration create_attachments_table --create=attachments`

마이그레이션 코드 작성  
```php
public function up()
{
    Schema::create('attachments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id');
        $table->string('attachment_type');
        $table->integer('attachment_id');
        $table->string('url');
        $table->timestamps();
        $table->softDeletes();
        $table->index(['deleted_at', 'attachment_type', 'attachment_id']);
    });
}
```

마이그레이션  
`artisan migrate`

모델 생성  
`artisan make:model Attachment`

모델 작성  
```php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Attachment extends Model
{
    //
    use SoftDeletes;

    protected $table = "attachments";
    protected $dates = ['deleted_at'];
}
```

`PostController.php` 에 모델 연결  

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

use App\Post;
use App\Attachment;
```

스토리지 링크 연결  

`artisan storage:link`

> 윈도우 환경에서는 cmd를 관리자 권한으로 실행시킨 후 vagrant에 접속하여야 된다.  

`post@store`, `post@update` 에 첨부 파일 코드 추가  

```php
public function store(Request $request){
    //
    $validatedData = $request->validate([
        'title' => 'required|max:255',
        'body' => 'required',
        'thumbnail' => 'image|mimes:jpeg,png,jpg,gif,svg|max:2048'
    ]);

    $post = new Post();
    $post->title = $request->title;
    $post->body = $request->body;
    $post->user_id = Auth::id();
    $post->save();

    if($request->hasFile('thumbnail')){
        $path = $request->file('thumbnail')->store('public/upfiles/board');

        $attachment = new Attachment;
        $attachment->user_id = Auth::id();
        $attachment->attachment_id = $post->id;
        $attachment->attachment_type = 'posts';
        $attachment->url = str_replace("public", "storage", $path);
        $attachment->save();
    }

    return redirect()->route('posts.show', ['post' => $post->id]);

}
```

`Post` 모델과 `Attachment` 모델 관계 설정  

`Post`와 `Attachment`의 관계를 1:N 으로 할지 1:1로 할지가 고민이 되었지만...  
현재 폼에서는 1개의 사진만 등록/수정을 할수 있게 되어있으므로 1:1로 정함  
<br>

`Post` 모델에 attachment 관계 설정  

```php
class Post extends Model {
    //
    use SoftDeletes;

    protected $table = "posts";
    protected $dates = ['deleted_at'];

    public function attachment(){
        return $this->hasOne('App\Attachment', 'attachment_id')->where('attachment_type', 'posts');
    }

}
```

`post@index`, `post@show` 에 `attachment` 불러오기  

```php
public function show($id){
    //
    $post = Post::with(['attachment'])->find($id);

    echo "<pre>";
    print_r($post->toArray());
    echo "</pre>";

    return view('posts.show', compact('post'));
}
```

`Post` 모델에 `with` 라는 함수를 이용하여 불러왔다.  
이 밖에도 공식 문서를 보면 다른 방식으로도 불러올수 있지만... 난 이상하게 저렇게 불러오는 것을 선호한다.  

```
Array
(
    [id] => 2
    [user_id] => 1
    [title] => hello world
    [body] => 사진 업로드
    [created_at] => 2020-06-17T11:54:14.000000Z
    [updated_at] => 2020-06-17T11:54:14.000000Z
    [deleted_at] => 
    [attachment] => Array
        (
            [id] => 1
            [user_id] => 1
            [attachment_type] => posts
            [attachment_id] => 2
            [url] => storage/upfiles/board/qdnS3HldrP2ikxzGRUeD51WiCnhKXF57kMFmUE1T.jpeg
            [created_at] => 2020-06-17T11:54:14.000000Z
            [updated_at] => 2020-06-17T11:54:14.000000Z
            [deleted_at] => 
        )

)
```

`toArray`를 이용하여 데이터를 배열로 바꿔서 데이터를 확인해보면 위와 같이 나온다.