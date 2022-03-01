                  ---
title: "HTTP GET 메소드와 Body Request"
excerpt_separator: "<!--more-->"
categories:
  - development
tags:
  - REST API
  - HTTP Method
  - Body Request
---

현재 진행 중인 프로젝트를 하면서 여러가지 API 를 개발하고 있던 중 한가지 알게된 사실에 대해 기록을 하고자 포스팅을 합니다.

다름 아닌, GET Method API 사용 시 Body Request 로 Json 요청 값을 넣은 API 에 대한 이야기 입니다.

<!--more-->

### 이슈 사항
프로젝트가 시작되고 여러 API 를 로컬에서 개발하고, 테스트를 맞힌 후 테스트 서버에 배포를 하였는데, 400 에러가 발생하고 하는 현상이 나타났습니다.

로컬에서는 아무 문제 없이 잘돌던 소스가 왜 400 에러가 발생하지? 더군다나 단순 조회 API 로 몇가지 조회 조건을 json 으로 넘기는게 다인 API 였는데..

- 테스트 환경은 AWS 로 구성이 되어 있으며, 에러 메시지는 Cloud Front 에서 에러 페이지를 리턴 하고 있었습니다.

### 원인
제가 짠 소스와 에러 페이지 문구를 잘 살펴보면서, 특이사항(?) 이라고 생각되는 부분이 보였습니다. 

GET Method 를 사용하는 API 에서 Path 파라미터나 Query 파라미터가 아닌 Body Request 만 사용하는 API 를 만든 것이였습니다.

혹시나 하는 마음에 Query 파라미터로 받을 수 있게 소스를 수정 후 배포 했더니 정상적으로 API 응답값이 호출 되는 것이였습니다.

### 확인
우선 같이 일하는 동료에게 물어보니, GET Method 를 사용하면서 Body Request 를 사용하는게 일반적이진 않은 거 같다는 답변을 들었고, 그 부분에 대해 구글링을 통해 조사를 해보았습니다.

로컬에서는 되지만, 테스트 서버에서 되지 않는 이유가 이해가 되지 않았으나, 확인해본 결과 `실제 HTTP 규격에 제한은 없으나`, 

설계적인 측면에서 API 를 받아주는 `서버나 호출하는 클라이언트 종류에 따라, GET Method 에 Body Request 사용 시 에러를 뱉어 내는 경우가 있다`고 합니다.

그동안은 운이 좋게 에러를 뱉지 않는 웹 서버들을 만났나 봅니다. 🤔😂

관련 자세한 내용은 아래 참고한 블로그에 상세히 기록이 있으니, 꼭 읽어보시기 바랍니다.

### 마무리
이런 사소한 것을 모르고 10년 넘게 개발자로 일했다는게 부끄럽네요..😅

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://if1live.github.io/posts/http-get-request-with-body-and-http-library/](https://if1live.github.io/posts/http-get-request-with-body-and-http-library/)
[https://brunch.co.kr/@kd4/158](https://brunch.co.kr/@kd4/158)