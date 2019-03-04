---
layout: post
categories: articles
#author: mint
title: "jekyll & Github을 이용한 정적 사이트 구축"
excerpt: "jekyll 설치 및 Github 정적사이트 구축"
date: 2019-02-27 01:05:27
modified: 2019-03-04 01:05:27
tags: [jekyll,blog]
image:
  feature: 2019-02-28-jekyll-install-onUbuntu/jekyll02.png
share: true
sitemap: true
---
Jekyll은 정적 웹사이트(static website)생성기 입니다.  
간단한 소개페이지, 기술블로그 등 제한된 용도로 사용하기는 충분합니다.

## Jekyll 설치
#### 설치 환경

```console
mint@mint:~$ uname -a
Linux mint 4.15.0-45-generic #48-Ubuntu SMP Tue Jan 29 16:28:13 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
mint@mint:~$ cat /etc/issue
Linux Mint 19.1 Tessa \n \l
```

#### Ruby 및 필요 패키지 설치
 - sudo apt-get install patch zlib1g-dev liblzma-dev
 - sudo apt-get install ruby ruby-dev build-essential
 - Ruby 환경 설정

```console
$ ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux-gnu]

$ echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
$ echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
$ echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc
```

#### jekyll 과 bundler 설치하기
- gem install jekyll bundler

```console
mint@mint:~$ gem install jekyll bundler
Fetching: public_suffix-3.0.3.gem (100%)
Successfully installed public_suffix-3.0.3
Fetching: addressable-2.6.0.gem (100%)
Successfully installed addressable-2.6.0
Fetching: colorator-1.1.0.gem (100%)

...(생략)

Successfully installed bundler-2.0.1
Parsing documentation for bundler-2.0.1
Installing ri documentation for bundler-2.0.1
Done installing documentation for bundler after 3 seconds
26 gems installed

$ jekyll -v
jekyll 3.8.5
mint@mint:~$ bundle -v
Bundler version 2.0.1

```

#### 블로그 생성
- jekyll new testblog

```console
$ jekyll new testblog
Running bundle install in /home/mint/myblog/testblog...
  Bundler: Fetching gem metadata from https://rubygems.org/...........

 ...(생략)

  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java`.
New jekyll site installed in /home/mint/local_backup/gogo/12.study/myblog/testblog.

$ cd testblog/
mint@mint:~/testblog$ bundle exec jekyll serve
Configuration file: /home/mint/myblog/testblog/_config.yml
            Source: /home/mint/myblog/testblog
       Destination: /home/mint/myblog/testblog/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.372 seconds.
 Auto-regeneration: enabled for '/home/mint/myblog/testblog'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
  ```
- 네트웍을 통해서 웹페이지를 접근하려면 H옵션 사용

```console
testblog$ bundle exec jekyll serve -H 192.168.1.88
```
#### 확인
![jekyllsite](/images/2019-02-28-jekyll-install-onUbuntu/jekyll01.png 'site'){: width="100%" height="100%"}{: .center}

####  Github 페이지 생성 및 jekyll 기본 페이지 Push
- github 에 가입 후 프로젝트를 생성한다.  
    - create a new Repositories : [링크](https://github.com/new)
    - username.github.io 로 생성해야 주소를 사용할 수 있다.

![github](/images/2019-02-28-jekyll-install-onUbuntu/github01.png 'site'){: width="100%" height="100%"}{: .center}

- 생성한 Repository에 jekyll로 생성한 testlog를 올려준다.
    - git 작업을 위한 환경 설정(메일주소, 이름)과 초기화

```shell

# 블로그 폴더 생성
blog$ mkdir keneich.github.io
blog$ cd keneich.github.io

# git 사용을 위한 기본 환경 설정
keneich.github.io$ git config --global user.email "keneich2@gmail.com"
keneich.github.io$ git config --global user.name "keneich"

# git 사용을 위한 초기화
keneich.github.io$ echo "# keneich.github.io" >> README.md
keneich.github.io$ git init
keneich.github.io/.git/ 안의 빈 깃 저장소를 다시 초기화했습니다

# jekyll 기본 페이지 copy
keneich.github.io$ cp -r ../testblog/* .
keneich.github.io$ ls
404.html  Gemfile  Gemfile.lock  README.md  _config.yml  _posts  _site  about.md  index.md

# git을 사용한 파일 Push
keneich.github.io$ git status
keneich.github.io$ git add *
keneich.github.io$ git commit -m "first commit"
keneich.github.io$ git remote add origin https://github.com/keneich/keneich.github.io.git
keneich.github.io$ git push -u origin master

```

####  Github 에 생성된 페이지 확인
![github](/images/2019-02-28-jekyll-install-onUbuntu/github03.png 'site'){: width="100%" height="100%"}{: .center}


## Reference
* [https://jekyllrb.com/](https://jekyllrb.com/)

<!--
인용구
> 문구
테두리(코드)
`문구`
볼드(굵은
**문구**
강조
==문구==
기울기
*문구*
취소선
~~문구~~
밑줄
++밑줄++
윗첨자,아래첨자
동해불과 <sub>백두산이</sub> 마르고 닳도록 ^하느님이 보우
각주
APT[^APT] 공격은 공격 기술이 아닌 흐름으로
[^APT]: APT는 Advance, Persistance, Threat의 두문어로
코드블럭
```javascript
 aaa
```
글짜색
<span style="color:red">내용</span>

테이블
|column|column|column|
|:-----|:-----:|-----:|
|left|center|right|
이미지 삽입
![title](/images/2019-02-27-100-day-planning/20190227_162449.jpg '메모'){: width="100%" height="100%"}{: .center}
링크
[이름](주소)

## Reference
* [https://pivotal.io/cicd](https://pivotal.io/cicd)
-->
