---
layout: post
title: "라라벨(laravel) 7.x 시작하기 (2)"
author: "sjahn"
categories: php
tags: [php, laravel]
image: laravel_logo.jpg
---

<br>
### 라라벨(laravel) 7.x 시작하기 (2)
<br>
[이전글](/php/php-laravel-started.html)에서는 라라벨 설치를 위한 로컬 개발환경을 구축했다면,  


해당 페이지에서는 composer를 이용한 라라벨 설치 및 기본 설정을 기록한다.


#### 라라벨(laravel) 설치
1. vagrant 실행: `vagrant up`
2. vagrant 쉘 접속: `vagrant ssh`
3. 프로젝트 폴더로 이동: `cd Code`
4. 라라벨 설치

```sh
# 최신 버전 설치
composer create-project --prefer-dist laravel/laravel sjahn.homestead.test

# 특정 버전 설치
composer create-project --prefer-dist laravel/laravel 프로젝트명 "5.8.*"
```


#### 패키지 설치
1. 프로젝트 폴더로 이동: `cd sjahn.homestead.test`
2. 패키지 설치: `composer install`
3. 패키지 업데이트: `composer update`


위의 과정이 끝나면 브라우저에서 `http://sjahn.homestead.test` 접속을 한다.
설치 과정에서 문제가 없다면 정상적으로 라라벨 기본 페이지가 나온다.


#### `.env` 파일 내 database 설정
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

homestead 데이터 베이스를 이용하려면 위처럼 설정을 해주면 된다.  
클라이언트 프로그램으로 데이터베이스에 연결하려면 33060 포트로 연결하면 접속이 된다.


2020.06.12 현재 mac 환경에서 workbench로 접속 시  
`Protocol mismatch; server version = 11, client version = 10` 라는 에러를 뱉으며 클라이언트 접속이 안되어서 ssh 터널링을 이용해서 접속을 하였다.  
로컬 환경에서 ssh 터널링이라니... 많이 황당했지만... 일단 접속은 잘되는 것을 확인하였다.

```
SSH HOST:   192.168.10.10
SSH User:   vagrant
SSH Key:    ~/.ssh/id_rsa
SSH PORT:   (empty)

DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

데이터베이스 설정이 끝났으면, 이제 마이그레이션을 할 차례이다.  
<br>
`artisan migrate`
<br>

#### 회원모듈 패키지 설치  
기존에는 `artisan make:auth`를 이용하면 되었는데, 6.x부터 사라진듯 하다.  
그런데 `users` 테이블도 기본 마이그레이션에 있던데 왜 패키지로 분리된건지 모르겠다.

1. laravel/ui 패키지 설치: `composer require laravel/ui`
2. auth 패키지 설치: `artisan ui bootstrap --auth`
    > bootstrap 외에 vue, react도 있지만 이번 프로젝트에는 js 사용을 최대한 안할 예정이라서 pass 했다.
3. npm 패키지 설치 및 실행: `npm install && npm run dev` 

브라우저에서 `http://sjahn.homestead.test`을 새로고침하면 우측 상단에 login, register 메뉴가 추가된 것을 확인할 수 있다.  

다음에는 회원 테이블에 `last_login_at` 이라는 컬럼을 추가하고, 로그인할때마다 업데이트 되는 기능을 구현, 그리고 `posts` 테이블과 게시판을 만드는 작업을 할 예정이다.