---
layout: post
title: "pip mysql client 설치 시 에러"
author: "sjahn"
categories: python
tags: [python, pip, mysql-client]
image: iu_thumbnail.jpg
---

### pip mysql-client 설치 시 에러가 발생할 때

정확하게는 특정 에러 밖에 해결 못하지만  

내 경우에는 거의 90% 확률로 mysql-client 모듈 설치 에러가 해결되었던 것 같다.  

리눅스 환경에서는 주변에 기기가 없어서 해보질 못했다.  

sqlalchemy 혹은 django-orm 을 설치하는 경우에 필요한데  

만약에 다른 방법을 이용하고자 한다면  

다른 모듈인 [pymysql](https://pypi.org/project/PyMySQL/) 커넥터를 이용하는 방법도 있긴하다.



#### Mac 에서 설치

`brew`가 설치되어있다는 전제 하에 아래 코드를 실행한다.  
제일 중요한 부분이 마지막 부분에 `LDFLAGS` 부분

1. `brew install openssl`
2. `cd ~/path/to/your/project`
3. `LDFLAGS=-L/usr/local/opt/openssl/lib pip install mysqlclient`


#### Windows 환경에서 설치

1. [링크](https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysqlclient)에서 자신의 python 버전과 cpu 버전에 맞는 파일 다운로드
2. `cd ~/path/to/your/project`
3. 1에서 다운받은 파일로 설치

  `Ex. pip install D:\workspace\path\mysqlclient-1.4.6-cp37-cp37m-win32.whl`
