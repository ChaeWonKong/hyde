---
title: "React 앱을 PWA로 배포하기"
date: "2019-06-30"
categories:
  - React
  - PWA
excerpt: |
  React 앱을 PWA(Progressive Web App)로 배포해 본다. 
feature_text: |
  ## React 앱을 PWA로 배포하기
  React 앱을 PWA(Progressive Web App)로 배포해 본다. 
feature_image: "https://images.unsplash.com/photo-1556157382-4e063bb26661?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1500&q=80"
image: "https://images.unsplash.com/photo-1556157382-4e063bb26661?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1500&q=80"
---

## 목차


## PWA 왜 써야 하는가
스마트폰이 등장한 이래 빠른 속도로 웹은 데스크톱에서 모바일로 옮겨 갔다. 정부 자료에 따르면 만 3세 이상 인구의 88.5%가 모바일 인터넷 이용자다. ([인터넷 이용자 실태조사](https://www.msit.go.kr/cms/www/m_con/news/report/__icsFiles/afieldfile/2018/01/31/2017%20%EC%9D%B8%ED%84%B0%EB%84%B7%EC%9D%B4%EC%9A%A9%EC%8B%A4%ED%83%9C%EC%A1%B0%EC%82%AC%20%EC%9A%94%EC%95%BD%EB%B3%B4%EA%B3%A0%EC%84%9C(%EC%B5%9C%EC%A2%85).pdf), 과학기술정보통신부, 2018)

미국에서는 이미 2016년 모바일 웹 트래픽이 데스크톱 웹 트래픽을 앞질렀다.([웹도 모바일시대…PC 트래픽 첫 추월](http://www.zdnet.co.kr/view/?no=20161102090942&from=Mobile), ZDNetKorea, 2016)

많은 사용자가 모바일로 검색을 하고, 모바일로 웹사이트에 접속한다. 이러한 현상은 연령대가 어릴수록 두드러진다.

모바일 시대의 개막과 함께 네이티브 앱도 폭발적으로 증가했다. 반응형/적응형 웹을 구현하더라도 네티이브 앱 수준의 기능을 제공하지 못하는 웹의 한계로 사용자들은 비교적 단순한 어플리케이션도 웹이 아닌 네이티브 앱을 통해 이용해야만 했다.

구글 등에서 PWA라는 개념을 들고 나왔다. PWA, Progressive Web App의 줄임말이다. 웹이지만 홈 화면에 앱처럼 설치가 가능하다. 추가적인 캐시 스토리지 용량을 배당받기에 네이티브 앱처럼 정적인 자산들을 클라이언트에 저장해 이용할 수 있다. 아직 iOS에서는 한계가 있지만 네이티브 앱처럼 푸시 알림도 할 수 있다.

그렇다면 PWA의 장점은 무엇인가? **반응형 웹으로 제작한 사이트를 큰 노력 없이 모바일 앱과 유사한 기능을 하도록 개선할 수 있다는 것**이다. 어차피 많은 서비스들이 자체 홈페이지를 가지고 있고, 홈페이지에서 모바일 대응을 하고 있다. 이것을 큰 돈 들여 네이티브로 만들지 않아도, PWA를 활용해 모바일 앱처럼 사용자가 다운받아 쓸 수 있게 할 수 있다.

무엇보다 PWA로 전환하는 방법만 알면, **전환에 드는 노력은 거의 제로에 가깝다** serviceWorker와 manifest.json을 작성하고 앱 아이콘과 랜딩화면 정도를 디자인해두면 사실상 완성이다. 수천만원 이상이 드는 모바일 네이티브 앱 개발에 비해 훨씬 저렴하며, 개발 자체도 훨씬 빠르다.

무엇보다 Android와 iOS에서 모두 이용 가능하다. 물론 iOS에서는 몇가지 한계가 있긴 하다. 그러나 매우 저렴한 비용으로 기존의 웹 사이트를 모바일용 앱으로 전환할 수 있다는 것은 결코 작지 않은 메리트 임이 분명하다.

## React 프로젝트를 PWA로 전환하기
혹시 프론트엔드에서 React를 사용하고 있는가?

혹시 CRA로 프로젝트를 작성하였는가?

그렇다면 너무 간단하다. 10분만 투자하면 당신의 앱을 설치까지 할 수 있도록 만들 수 있다. 

## PWA의 조건