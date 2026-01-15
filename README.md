# Postman User-Agent 헤더 설정 및 변경

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 앤チボット 감지를 피하고 HTTP 요청를 개선하기 위해 Postman에서 User-Agent 헤더를 설정, 변경 및 로ーテーション하는 방법을 설명합니다:

- [Postman 기본 User Agent란 무엇입니까?](#what-is-the-postman-default-user-agent)
- [Postman User Agent 변경](#change-the-postman-user-agent)
  - [단일 요청에 User Agent 설정](#set-the-user-agent-on-a-single-request)
  - [전체 컬렉션에 User Agent 설정](#set-the-user-agent-on-an-entire-collection)
  - [User Agent 해제](#unset-the-user-agent)
- [Postman에서 User Agent ローテーション 구현](#implement-user-agent-rotation-in-postman)
  - [User Agent 목록 가져오기](#retrieve-a-list-of-user-agents)
  - [무작위로 User Agent 선택](#randomly-pick-a-user-agent)
  - [User-Agent 헤더 정의](#define-the-user-agent-header)
  - [전체 구성](#put-it-all-together)

## 왜 커스텀 User Agent를 설정해야 합니까?

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) 헤더는 HTTP 요청를 수행하는 클라이언트를 식별합니다. 일반적으로 클라이언트의 머신과 요청에 사용된 애플리케이션에 대한 세부 정보를 포함합니다. 웹 브라우저, [HTTP 클라이언트](https://brightdata.co.kr/blog/web-data/best-python-http-clients) 및 기타 소프트웨어는 이 헤더를 자동으로 설정합니다.

다음은 웹 페이지를 요청할 때 Chrome이 사용하는 `User-Agent` 문자열의 예입니다:

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
```

브라우저의 `User-Agent` 헤더는 여러 핵심 구성 요소로 이루어집니다:

- **`Mozilla/5.0`** – 원래는 Mozilla 호환성을 나타냈으며, 현재는 더 넓은 지원을 위해 포함됩니다.  
- **`Macintosh; Intel Mac OS X 10_15_7`** – 운영 체제(`Mac OS X 10.15.7`) 및 플랫폼(Intel Mac)을 나타냅니다.  
- **`AppleWebKit/537.36`** – Chrome이 사용하는 렌더링 엔진을 지정합니다.  
- **`(KHTML, like Gecko)`** – KHTML 및 Gecko 레이아웃 엔진과의 호환성을 보장합니다.  
- **`Chrome/127.0.0.0`** – 브라우저 이름 및 버전을 나타냅니다.  
- **`Safari/537.36`** – Safari와의 호환성을 시사합니다.  

서버는 `User-Agent` 헤더를 사용하여 요청가 브라우저에서 온 것인지, 다른 소스에서 온 것인지 식별합니다.  

Web스크레이핑 봇에서 흔히 하는 실수는 기본값 또는 비브라우저 `User-Agent` 문자열을 사용하는 것이며, 앤チボット 보호는 이를 쉽게 감지하고 차단합니다.

## Postman 기본 User Agent란 무엇입니까?

[Postman](https://www.postman.com/)은 널리 사용되는 데스크톱 HTTP 클라이언트입니다. 대부분의 HTTP 클라이언트와 마찬가지로, 각 요청에 기본 `User-Agent` 헤더를 자동으로 설정합니다.

Postman에서 자동 생성된 헤더를 검사하면 이 동작을 확인할 수 있습니다.

![observe this behavior by examining the auto-generated Postman headers](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/observe-this-behavior-by-examining-the-auto-generated-Postman-headers-1024x396.png)

보시는 것처럼 Postman 기본 user agent는 다음 형식을 따릅니다:

```
PostmanRuntime/x.y.z
```

`PostmanRuntime/x.y.z` 문자열은 Postman이 수행한 요청임을 나타내며, `x.y.z`는 버전 번호를 의미합니다.

이를 확인하려면 [`httpbin.io/user-agent`](https://httpbin.io/user-agent)로 GET 요청를 전송하십시오. 이 엔드포인트는 들어오는 リクエ스트의 `User-Agent` 헤더를 반환하므로, 어떤 HTTP 클라이언트가 사용한 user agent인지 검증할 수 있습니다.

![identify the user agent used](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/identify-the-user-agent-used-1024x593.png)

API가 반환한 user agent가 Postman이 기본으로 설정한 값과 일치하는 것을 확인할 수 있습니다. 구체적으로 Postman user agent는 다음과 같습니다:

```
PostmanRuntime/7.41.0
```

Postman의 `User-Agent`는 브라우저 헤더와 다르므로, 앤チボット 시스템에 의해 요청가 차단될 가능성이 높아집니다. 이러한 시스템은 비정상적인 user agent를 포함해 봇 같은 패턴을 감지합니다. 기본 `User-Agent`를 변경하면 감지를 피하는 데 도움이 됩니다.

## Postman User Agent 변경 방법

### 단일 요청에 User Agent 설정

Postman에서는 `User-Agent` 헤더를 수동으로 지정하여 단일 HTTP 요청에서 user agent를 변경할 수 있습니다.

> **참고**:
> 
> Postman에서 자동 생성되는 헤더는 직접 수정할 수 없습니다.

“Headers” 탭으로 이동한 다음 새 `User-Agent` 헤더를 추가하십시오:

![adding a new user-agent header](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/adding-a-new-user-agent-header-1024x333.gif)

Postman은 기본 `User-Agent`를 사용자 정의 헤더로 대체합니다. [HTTP 헤더는 대소문자를 구분하지 않으므로](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) `User-Agent`, `user-agent` 또는 다른 변형을 사용할 수 있습니다.

`httpbin.io/user-agent`로 GET 요청를 보내 변경 사항을 확인하십시오.

![executing a GET request](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/executing-a-GET-request-1024x578.png)

### 전체 컬렉션에 User Agent 설정

[Postman 컬렉션](https://www.postman.com/collection/)은 공유 구성이 적용된 API 요청의 그룹입니다. Postman에서는 컬렉션 내 각 요청 전후에 실행되는 [커스텀 스크립트](https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/)를 정의할 수 있습니다.

모든 요청에 커스텀 `User-Agent`를 적용하려면 Postman 계정에 로그인하거나 [계정을 생성](https://www.postman.com/postman-account/)하십시오.  

정리된 HTTPBin 엔드포인트를 가진 "HTTPBin" 컬렉션이 있다고 가정합니다:

![HTTPBin endpoints organized in folders](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/HTTPBin-endpoints-organized-in-folders-1024x554.png)

`/user-agent` 엔드포인트에 대한 요청를 실행하면 기본 Postman user agent가 반환됩니다:

![the default Postman user agent](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/the-default-Postman-user-agent-1024x555.png)

[pre-request 스크립트](https://learning.postman.com/docs/tests-and-scripts/write-scripts/pre-request-scripts/)는 Postman 컬렉션에서 각 요청 전에 실행되는 JavaScript 함수입니다. 이를 사용해 커스텀 `User-Agent` 헤더를 설정할 수 있습니다.

pre-request 스크립트를 만들려면:  
1. 컬렉션을 여십시오.  
2. **Scripts** 탭으로 이동하십시오.  
3. **Pre-request** 옵션을 선택하십시오.  

![select the “Pre-request” option](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/select-the-Pre-request-option-1024x557.gif)

에디터에 다음 코드를 붙여넣으십시오:

```js
pm.request.headers.add({

key: "User-Agent",

value: "<your-user-agent>"

});
```

아래와 같이 `<your-user-agent>` 문자열을 사용하려는 user agent 값으로 바꾸십시오:

```js
pm.request.headers.add({

key: "User-Agent",

value: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36"

});
```

[`pm.request.headers.add()`](https://learning.postman.com/docs/tests-and-scripts/write-scripts/postman-sandbox-api-reference/)는 Postman API에서 제공하는 특수 함수로, 요청에 특정 헤더를 추가합니다.

“Save” 버튼을 클릭하여 변경 사항을 적용하십시오.

`/user-agent` 엔드포인트에 대한 요청를 다시 실행하십시오:

![executing the request again ](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/executing-the-request-again-1024x559.png)

이번에는 기본 Postman user agent가 아니라 스크립트에서 설정한 user agent가 반환됩니다.

### User Agent 해제

자동 생성되는 `User-Agent` 헤더는 선택 사항이며, 실제로 체크를 해제할 수 있습니다. 체크를 해제하면 Postman은 더 이상 `User-Agent` 헤더를 전송하지 않습니다.

들어오는 요청의 모든 헤더를 반환하는 [`httpbin.io/headers`](https://httpbin.io/headers) 엔드포인트로 요청를 보내 이를 확인하십시오:

![return of all the headers of the incoming request](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/return-of-all-the-headers-of-the-incoming-request-1024x640.png)

엔드포인트가 반환한 headers 객체에 `User-Agent` 키가 포함되지 않은 것을 확인할 수 있습니다.

> **참고**:
>
> 거의 모든 웹 요청에는 일반적으로 해당 헤더가 포함되므로, `User-Agent` 헤더 해제는 권장되지 않습니다.

## Postman에서 User Agent ローテーション 구현

Postman의 `User-Agent`를 브라우저 문자열로 단순히 교체하는 것만으로는, 특히 동일한 IP 주소에서 반복 요청를 수행하는 경우 앤チボット 시스템을 우회하지 못할 수 있습니다.

감지를 피하려면 각 요청에 서로 다른 `User-Agent`를 할당하는 _user agent ローテーション_을 사용하십시오. 이렇게 하면 봇으로 플래그될 가능성이 줄어듭니다.

이제 Postman에서 user agent ローテーション을 구현하는 방법을 알아보겠습니다.

### User Agent 목록 가져오기

[WhatIsMyBrowser.com](https://www.whatismybrowser.com/guides/the-latest-user-agent/) 같은 사이트를 방문하여 유효한 user agent 목록을 채우십시오:

```js
const userAgents = [

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Linux; Android 10; HD1913) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.103 Mobile Safari/537.36 EdgA/127.0.2651.82",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (iPhone; CPU iPhone OS 17_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/127.0.6533.107 Mobile/15E148 Safari/604.1",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14.6; rv:129.0) Gecko/20100101 Firefox/129.0",

// other user agents...

];
```

> **팁**:
> 
> 이 배열에 실제 환경의 user agent가 더 많이 포함될수록, ローテーション 가능성이 증가하므로 더 좋습니다.

### 무작위로 User Agent 선택

JavaScript [`Math`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) API를 사용하여 목록에서 user agent를 무작위로 선택하십시오:

```js
const userAgent = serAgents[Math.floor(Math.random() * userAgents.length)];
```

이 코드 줄에서 수행되는 동작은 다음과 같습니다:

1.  [`Math.random()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)이 0과 1 사이의 난수를 생성합니다.
2.  생성된 숫자에 `userAgents` 배열의 길이를 곱합니다.
3.  [`Math.floor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor)가 결과 값을 내림하여, 소스 숫자보다 작거나 같은 가장 큰 정수로 만듭니다. 이 결과는 0부터 `userAgents.length - 1`까지의 인덱스에 해당합니다.
4.  해당 인덱스를 사용해 user agent 배열에서 임의의 항목에 접근합니다.
5.  무작위로 선택된 user agent를 변수에 할당합니다.

### User-Agent 헤더 정의

`pm.request.headers.add()`를 사용하여 무작위 `userAgent` 값으로 `User-Agent` 헤더를 정의하십시오:

```js
pm.request.headers.add({

key: "User-Agent",

value: userAgent

});
```

이제 컬렉션의 HTTP 요청에는 ローテーティング `User-Agent` 헤더가 포함됩니다.

### 전체 구성

다음은 user agent ローテーション을 위한 최종 Postman pre-request 스크립트입니다:

```js
// a list of valid user agents

const userAgents = [

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Linux; Android 10; HD1913) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.103 Mobile Safari/537.36 EdgA/127.0.2651.82",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (iPhone; CPU iPhone OS 17_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/127.0.6533.107 Mobile/15E148 Safari/604.1",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14.6; rv:129.0) Gecko/20100101 Firefox/129.0",

// other user agents...

];

// randomly extract a user agent from the list

const userAgent = userAgents[Math.floor(Math.random() * userAgents.length)];

// set the random user agent header

pm.request.headers.add({

key: "User-Agent",

value: userAgent

});
```

스크립트를 컬렉션에 추가한 다음 `httpbin.io/user-agent`로 요청를 전송하여 테스트하십시오. 여러 번 요청를 실행하여 로ーテーティング user agent가 실제로 동작하는지 확인하십시오.

![executing requests and seeing the rotation of user agents](https://github.com/bright-kr/postman-user-agent-header/blob/main/images/executing-requests-and-seeing-the-rotation-of-user-agents-1024x557.gif)

반환되는 user agent가 계속 변경됩니다.

## 결론

`User-Agent` 헤더를 오버라이드하고 user agent ローテーション을 구현하면 기본적인 앤チボット 메커니즘을 회피하는 데 도움이 됩니다. 그러나 더 고급 시스템은 여전히 요청를 차단할 수 있습니다. IP 밴을 방지하려면 [Postman에서 프록시를 사용](https://brightdata.co.kr/integration/postman)할 수 있습니다.

지금 가입하여 무료 체험을 시작하십시오.