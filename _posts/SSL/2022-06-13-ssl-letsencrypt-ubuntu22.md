---
layout: post
current: post
cover: assets/built/images/nginx-tomcat/nginx-logo.png
navigation: True
class: post-template
author: sian
title: "Letsencrypt + Nginx SSL인증서(HTTPS) 적용 (Ubuntu22.04 개정판)"
comments: true
categories:
  - SSL
tags:
  - https
  - SSL
  - letsencrypt
date: "2022-06-13"
---

## 서론

이전에 [ubuntu 18버전에 Let's Encrypt를 이용하여 SSL 환경을 구성한 포스트]를 작성했었다. 몇년이 지나고 현재 ubuntu 22 버전에 다시 SSL을 세팅할 일이 생겨서 이전에 작성한 포스트를 보며 변경된 부분을 개정하여 본 포스트를 작성하였다.

[ubuntu 18버전에 Let's Encrypt를 이용하여 SSL 환경을 구성한 포스트]: https://sianlab.github.io/ssl-letsencrypt/	"Letsencrypt + Nginx SSL인증서(HTTPS) 적용"



### 환경

- Ubuntu 22.04
- apt-get 을 통해 설치한 Nginx
- 도메인 ( 이전에는 ngrok를 이용한 임시 도메인을 사용하였지만 이번에는 실 도메인 사용 )



## Let's Encrypt

- 웹 사이트에 HTTPS (SSL / TLS)를 사용하기 위해 필요한 디지털 인증서를 무료로 제공
- 인증서의 자동 갱신을 지원
- 자세한 내용은  [ Let's Encrypt 공식 사이트](https://letsencrypt.org/about/) 에서 확인



## Nginx 설치

~~~bash
$ sudo apt-get install nginx
~~~

80 포트를 사용하는 또다른 서비스가 존재한다면 서비스를 중지하거나 삭제 한 후 nginx 설치

설치완료 후 간단하게 curl을 이용하여 웹사이트가 오픈되었는 지 확인해보았다. curl을 사용하기 어려운 환경이라거나 브라우저로 직접 확인해보고 싶다면 localhost 주소에 http 기본 포트인 80 포트로 접속해보면 된다. 

![img](\assets\built\images\letsencrypt\ubuntu22\ssl1.jpg)

![img](\assets\built\images\letsencrypt\ubuntu22\ssl2.jpg)

Nginx 화면이 나오지 않을 시에는

~~~bash
$ sudo service nginx restart
~~~

명령어로 nginx 를 재시작 해본 뒤 다시 웹 브라우저에서 접속시도 nginx index 화면이 제대로 나온다면 이제 Let's Encrypt 를 이용하여 SSL 인증서를 받아보자



## certbot  설치

이전에 작성한 글을 따라 certbot 저장소를 추가하려고 했는데 아래와 같이 deprecated 됬다고 한다.

![img](\assets\built\images\letsencrypt\ubuntu22\ssl3.jpg)

그래서 [certbot install 사이트]를 참고하여 certbot을 설치하였다.

[certbot install 사이트]: https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal/ "certbot install 사이트"

certbot 사이트에 따르면 certbot 설치 시 이제는 snap을 이용하는 것으로 보인다. snap은 ubuntu에 기본적으로 설치되어 있다고 하니 따로 설치하지 않고 바로 명령어를 이용하여 certbot을 설치하였다. (ubuntu 어느 버전부터 기본 설치가 되어 있는지는 확인하지 못하였다.)

certbot 설치 전에 snap을 최신화해주자.

~~~bash
$ sudo snap install core; sudo snap refresh core
~~~

다음으로 certbot을 설치한다.

~~~bash
sudo snap install --classic certbot
~~~

설치 완료 시 아래와 같은 화면이 나왔다.

![img](\assets\built\images\letsencrypt\ubuntu22\ssl4.jpg)

다음으로 certbot 명령어를 바로 사용할 수 있게 아래와 같이 링크를 만들어준다.

~~~bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
~~~

아래와 같이 아무 에러도 없다면 링크파일 생성에 성공한 것이다.

![img](\assets\built\images\letsencrypt\ubuntu22\ssl5.jpg)

이제 certbot을 이용하여 인증서를 내려받고 nginx에 연동을 한다. 설치 가이드에 따르면 인증서 내려받기 및 nginx 연동까지 자동으로 해주는 방법과 그냥 인증서만 내려받는 방법 두가지가 소개되어 있는데, nginx도 기본으로 설치하고 따로 세팅한 부분이 없으므로 nginx 설정까지 자동으로 해주는 명령어를 이용해보았다.

아래 명령어를 입력하자.

~~~bash
sudo certbot --nginx
~~~

그러면 아래와 같이 이메일과 자신의 도메인을 입력하면 certbot이 자동으로 환경을 구성해준다.

![img](\assets\built\images\letsencrypt\ubuntu22\ssl6.jpg)

인증서와 키파일의 저장 경로도 표시해주니 필요 시 따로 기록해두면 될 것 같다.



## 인증서 갱신

certbot이 인증서의 자동 갱신도 지원해주기 때문에 갱신에 관해 별도로 신경쓰지 않아도 되는 것으로 보인다. 가이드 페이지에 갱신 테스트 명령어도 있으니 필요한 사람은 직접 입력해보면 될 것 같다.



## HTTPS 확인

발급 받은 인증서가 제대로 인증되는 지 확인을 위해 이용중인 브라우저로 내 도메인 사이트에 접속해본다. 아래와 같이 URL 표시줄에 자물쇠 표시가 나타나면 성공한 것이다.

![img](\assets\built\images\letsencrypt\ubuntu22\ssl7.jpg)



## troubleshooting

본 포스트 내용을 진행중에 따로 에러가 난 부분이 없어서 추후 에러사항이 있다면 추가해나갈 계획이다.