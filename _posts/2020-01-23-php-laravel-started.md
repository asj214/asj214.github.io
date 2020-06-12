---
layout: post
title: "라라벨(laravel) 7.x 시작하기 (1)"
author: "sjahn"
categories: php
tags: [php, laravel]
image: laravel_logo.jpg
---

### 라라벨(laravel) 7.x 시작하기 (1)
일전에 5.8 버전으로 작업을 해본 경험이 있었는데, 어느새 6.x를 지나고 7.x까지 온 듯하다.  

최근에 개발 언어를 python으로 변경하면서 php쪽은 살짝 소외되었었는데, 공부용으로라도 기록해야할듯하여 처음부터 다시 진행해본다.

나는 Windows, Mac 두가지를 둘다 사용하므로 최대한 환경의 영향이 없는 homestead를 다시 이용하기로 했다.  

#### 준비물
1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or VMWare
2. [Vagrant](https://www.vagrantup.com/downloads.html)
3. git


#### 설치 시 도움이 많이 되었던 페이지
1. [라라벨 코리아 / 홈스테드](https://laravel.kr/docs/7.x/homestead)
2. [라라벨 vagrant 설치, 설정 등 좀 더 자세한 설명 페이지](https://dev-sunny-jinny.tistory.com/3)
3. [가상호스트 추가 시 도움 될 페이지](https://stackoverflow.com/questions/31139284/using-vagrant-and-homestead-for-multiple-sites-and-per-project-installation)


#### vagrant box 설치
`vagrant box add laravel/homestead`


#### homestead 설치하기
```sh
# git clone
git clone https://github.com/laravel/homestead.git

# homestead folder 로 이동
cd Homestead

# release branch 변경
git checkout release

# Mac / Linux...
bash init.sh

# Windows...
init.bat
```

#### Homestead.yaml 작성하기
```yaml
---
ip: "192.168.10.10"
memory: 2048
cpus: 2
provider: virtualbox

authorize: ~/.ssh/id_rsa.pub

keys:
    - ~/.ssh/id_rsa

folders:
    - map: /Users/sjahn/workspace
      to: /home/vagrant/Code

sites:
    - map: sjahn.homestead.test
      to: /home/vagrant/Code/sjahn.homestead.test/public

databases:
    - homestead

features:
    - mariadb: false
    - ohmyzsh: false
    - webdriver: false

# ports:
#     - send: 50000
#       to: 5000
#     - send: 7777
#       to: 777
#       protocol: udp
```

사실 공식 사이트에 친절하게 잘나와있어서 크게 변경할 것은 없었다.  


자신의 로컬 개발 환경에 맞게 `folders`, `sites` 맞 설명해주면 문제가 없을 것 같다.  


#### hosts 파일에 추가
- Mac: sudo vi /etc/hosts
- Windows: C:\Windows\System32\drivers\etc\hosts

```
192.168.10.10	    sjahn.homestead.test
```


#### 자주 사용하는 vagrant command
```sh
# vagrant 실행
vagrant up

# vagrant 일시 중지
vagrant suspend

# vagrant 종료
vagrant halt

# 설정 파일 수정 후 적용
vagrant reload --provision
```
> `vagrant reload --provision` 의 경우에는 Homestead.yaml 파일을 수정하는 경우에 쓰인다.  

<br>
<br>
여기까지 진행했다면 초반 환경 구축은 거의 끝났다.  
글이 길어지므로 [다음글](/php/php-laravel-stated_2.html)에서 계속  