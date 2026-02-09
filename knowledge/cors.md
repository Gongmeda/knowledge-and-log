# CORS

진행중인 프로젝트에서 CORS 관련 이슈를 겪었습니다.

해당 프로젝트는 실제 배포를 목표로 진행하고 있는 프로젝트이기 때문에 좀 더 확실한 이해를 기반으로 문제를 해결하고 넘어가야겠다고 생각했습니다.

## 이슈 상황

![](https://velog.velcdn.com/images/gongmeda/post/819d2e68-2227-446c-a7e9-9f3ea072b742/image.png)

프론트엔드와 백엔드를 연결하는 과정에서 CORS 관련 이슈를 마주칠 수 있습니다.

`axios` 와 같은 HTTP 요청 라이브러리를 사용해서 백엔드 서버에 요청을 보냈을 때 서버는 정상적으로 요청을 처리하고 응답했지만 위와 같은 에러가 콘솔창에 뜨면서 브라우저가 API의 응답을 거절했을 것 입니다.

에러 메시지를 해석해보면 '`preflight 요청` 의 응답이 `Access-Control-Allow-Origin` 이라는 헤더를 포함하고 있지 않다' 는 뜻인데요, `preflight 요청` 이 무엇이고 이러한 헤더가 필요한 이유를 알기 위해 역사를 조금 알아보겠습니다.

## 과거의 웹과 SOP

과거에는 웹사이트를 만든다 하면 대부분 브라우저의 요청에 따라 서버에서 정적, 또는 동적으로 HTML을 렌더링해서 HTML을 통째로 응답해주는 방식이었습니다.

따라서, 필요한 비즈니스 로직은 모두 서버에서 처리되었기 때문에 필요에 따라 다른 주소로 요청을 보내는 행위를 할 필요성이 없었습니다. 또한, 출처가 다른 곳에 요청을 보내는 행위가 해커들에 의해 악용되는 경우(XSS, CSRF 등)가 있었기 때문에 이러한 행위를 비정상적인 행위라고 간주하고 막는 것을 기본적인 보안 정책으로 설정한 것이 바로 `동일 출처 정책` 또는 `SOP(Same-Origin Policy)` 입니다.

SOP는 어떤 출처에서 불러온 문서나 스크립트가 다른 출처에서 가져온 리소스와 상호작용하는 것을 제한하는 보안 정책입니다.

여기서 출처는 Origin을 뜻하며, URL에서 `프로토콜`, `포트`, `호스트` 를 기준으로 판단합니다. 세 가지 모두 같아야 동일한 출처로 판단합니다.

따라서 아래와 같은 경우들은 모두 SOP를 위반하는 것 입니다.

1. 프론트엔드를 `https://gongmeda.com` 에서 호스팅하고 `https://api.gongmeda.com` 으로 요청을 보내는 것
2. 프론트엔드를 `https://gongmeda.com` 에서 호스팅하고 `http://gongmeda.com/api` 로 요청을 보내는 것
3. 프론트엔드를 `http://localhost:3000` 에서 호스팅하고 `http://localhost:8080` 로 요청을 보내는 것

## CORS

하지만 웹이 발전하면서 HTTP 요청이 HTML이 아닌 XML, JSON 형태의 정보를 전달해주게 되었고, 프론트엔드에서 비즈니스 로직을 처리하는 경우가 늘면서 필요에 따라 출처가 다른 곳에 요청을 보내서 정보를 불러와야 하는 요구 사항이 생겼습니다.

현재 저희가 카카오, 페이스북, 트위터와 같은 제 3자의 API를 사용하는 것과 같은 것 말이죠.

하지만 SOP는 이 모든 행위를 막는 정책입니다. 그래서 `JSONP` 라고 하는 script 태그는 다른 출처에서 데이터를 불러오는 것이 가능한 허점을 활용하여 다른 출처와 데이터를 주고 받는 방식을 활용하기 시작했습니다.

하지만 이는 비정상적인 방식이었기 때문에 다른 출처로의 요청을 특정 조건하에 허용하기 위해 나온 정책이 바로 `교차 출처 리소스 공유` 또는 `CORS(Cross-Origin Resource Sharing)` 입니다.

CORS는 추가 HTTP 헤더를 사용하여, 한 출처에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제입니다.

**즉, SOP는 다른 출처로의 요청을 막는 정책이고, 이를 특정 조건에서는 허용하는 정책이 CORS인 것 입니다.**

### 접근 제어 시나리오

CORS의 정책에 의한 요청 시나리오는 총 3가지가 있습니다.

1. `Preflight Request`
2. `Simple Request`
3. `Credentialed Request`

#### Preflight Request

![](https://velog.velcdn.com/images/gongmeda/post/0c22dc94-37b6-4c9f-886a-c4952251a3c4/image.png)


해당 시나리오는 다음과 같은 순서로 동작합니다.

1. `OPTIONS` HTTP 메소드로 원래 보내려고 한 요청의 정보를 담아서 보냅니다. 이 요청을 `Preflight Request` 라고 합니다. 정보에는 `요청을 보내는 출처`, `요청에 사용할 HTTP 메소드`, `요청에 담을 헤더들` 이 있습니다.
2. `Preflight Request` 에 대해 서버는 허용하는 `출처`, `메소드`, `헤더`, `캐싱될 시간` 등을 담아서 응답합니다. 올바른 응답 코드는 200대여야 하며, `Access-Control-Allow-뭐시기` 하는 헤더들의 값이 모두 원래 요청에 대해 허용되는 값이어야 합니다.
3. 원래 요청을 보냅니다.
4. 서버는 원래 요청에 대한 응답을 보냅니다.

이제 우리는 위에서 만났던 콘솔의 에러 메시지를 해석할 수 있습니다.

![](https://velog.velcdn.com/images/gongmeda/post/819d2e68-2227-446c-a7e9-9f3ea072b742/image.png)

위 사진을 다시 해석하면

> `http://localhost:8080/test` 로 요청을 보내려고 했는데, 현재 페이지 Origin이랑 달라서 CORS 정책에 따라 `Preflight  Request` 를 보내봤다.
근데, 응답에 `Access-Control-Allow-Origin` 헤더가 없더라.
이게 없으면 어떤 출처에서 오는 요청을 허용하는 서버&경로인지 모르기 때문에 보안 정책상 안된다.

라는 의미가 됩니다.

#### Simple Request

![](https://velog.velcdn.com/images/gongmeda/post/53dd65bc-d4d2-47d7-82ac-54222d5b031b/image.png)

`Simple Request` 는 Preflight 없이 원래 요청, 그리고 원래 요청에 대한 응답만으로 구성되어 있습니다.

하지만 다음과 같은 조건을 만족해야 합니다.

1. `GET`, `POST`, `HEAD` 메소드만 가능
2. `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `Range` 헤더만 가능
3. `Content-Type` 헤더의 값은 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 만 가능

따라서 `Content-Type: application/json` 을 사용하는 REST API는 모두 Preflight 가 필요한 것 입니다.

직접 해보면 다음과 같습니다.

![](https://velog.velcdn.com/images/gongmeda/post/4274b965-d706-4a6d-9517-bda408a5141c/image.png)

![](https://velog.velcdn.com/images/gongmeda/post/3abc1e73-864e-44ca-a0a8-3b2516fdc4aa/image.png)


위와 같이 요청에 `Content-Type: application/json` 헤더가 없는 경우에는 요청이 Preflight 없이 `Simple Request` 하나만 보내지는 것을 확인할 수 있습니다.

![](https://velog.velcdn.com/images/gongmeda/post/22b7fed8-3ce0-46ec-9b01-75907e839b8d/image.png)

![](https://velog.velcdn.com/images/gongmeda/post/7d033c11-8406-4ba1-8610-6046ad0ee02d/image.png)

하지만 `Content-Type: application/json` 헤더가 추가되는 순간, `Simple Request` 의 제약 조건을 만족하지 않기 때문에 `Preflight Request` 로 처리되는 모습을 확인할 수 있습니다.

#### 왜 Preflight Request가 필요한가?

Simple Request를 알고나면 의문이 들 수 있습니다.

> 왜 Simple Request로 처리하지 않고 굳이 Preflight를 포함해서 2번 요청을 보내는 것일까?

Preflight는 CORS 정책을 모르는 서버를 위한 스펙입니다.

과거에는 위에서 언급한 것 처럼 SOP만 존재하던 시절이 있었고, 따라서 다른 출처에서 요청을 보내는 것에 대한 보안 메커니즘이 존재하지 않는 서버들이 그 시절에는 문제 없이 동작하고 있었을 것 입니다.

하지만, CORS로 인해 브라우저에서 다른 출처로의 요청이 허용되면서 이런 요청에 대한 보안 처리가 되어있지 않은 서버를 보호하기 위한 장치로서 이러한 방식이 채택된 것 입니다.

왜냐하면 OPTIONS 메소드로 요청 가능 유무를 확인하는 과정에서는 원래 요청을 보내지 않고, 따라서 원래 요청에 대한 비즈니스 로직이 실행될 여지도 없기 때문이죠.

가령 허용하면 안되는 출처에서 DELETE 요청을 보냈을 때, 삭제가 동작하면 안되지만 출처 검증 로직이 없어서 삭제되는 서버라면 Preflight로 막을 수 있을 것 입니다.

#### Credentialed Request

![](https://velog.velcdn.com/images/gongmeda/post/561a41da-65f0-4503-b38e-2c32f611471b/image.png)

`Credentialed Request` 는 인증된 요청을 보낼 때 사용되는 방식입니다.

주로 쿠키와 같은 정보를 포함해서 요청을 보내고 싶을 때 사용됩니다.

![](https://velog.velcdn.com/images/gongmeda/post/912873bf-c2ac-4b2b-b584-81ed7f3bc4a3/image.png)

`axios` 에서는 위와 같이 `withCredentials` 라는 옵션을 `true` 로 설정하면 요청에 쿠키가 포함되게 됩니다.

하지만 민감한 정보가 포함되는 만큼 추가적인 필수 조건들이 생깁니다.

1. 응답에 `Access-Control-Allow-Credentials: true` 헤더가 존재해야 한다.
2. `Access-Control-Allow-Origin` 의 값이 `*` 이면 안된다. (허용하는 출처를 명시적으로 작성해야 함)

다음은 위 조건들을 충족하지 않을 경우에 대한 크롬 브라우저의 오류들 입니다.

![](https://velog.velcdn.com/images/gongmeda/post/812c9dcd-5d55-4725-ad86-e0a1e858c3d1/image.png)

![](https://velog.velcdn.com/images/gongmeda/post/057a3cf4-05ec-4468-bc35-d83a178ac06f/image.png)

위는 `Access-Control-Allow-Credentials: true` 헤더가 존재하지 않기 때문에 생긴 에러메시지입니다.

![](https://velog.velcdn.com/images/gongmeda/post/207f5bb4-3d5b-4822-88af-4f818ad0b49e/image.png)

![](https://velog.velcdn.com/images/gongmeda/post/fba34308-1289-49f3-9b79-20d8e1560d26/image.png)

위는 `Access-Control-Allow-Credentials: true` 헤더가 존재하지만, `Access-Control-Allow-Origin` 의 값이 `*` 여서 생긴 에러메시지입니다.

## 참고

- https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
- https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- https://youtu.be/yTzAjidyyqs
- https://youtu.be/-2TgkKYmJt4