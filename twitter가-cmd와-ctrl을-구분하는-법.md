# Twitter가 cmd와 ctrl을 구분하는 법

Twitter Help Center의 [How to Tweet](https://help.twitter.com/en/using-twitter/how-to-tweet) 문서를 보자.

"send Tweet" 동작에 대한 단축키가 `cmd-enter` 혹은 `ctrl-enter`으로 지정되어 있는데, 실제로 macOS 운영체제에서는 커맨드 엔터, Windows 운영체제에서는 컨트롤 엔터에 의해서만 위 동작이 호출된다, 정확히 이것을 구분하고 있다. 이것을 어떻게 구분하고 있을까?

망글링 되어 있는 트위터 앱의 [스크립트 파일](https://abs.twimg.com/responsive-web/client-web/shared~bundle.RichTextCompose~bundle.DMRichTextCompose~ondemand.RichText.da96bfd5.js)을 보면, hasCommandModifier 플래그가 가장 유력한 후보자인데, 이것은 페이스북의 리액트 텍스트 에디터 [Draft.js](https://draftjs.org/) 내 [KeyBindingUtil](https://draftjs.org/docs/api-reference-key-binding-util/) 모듈의 정적 메소드 이름이다. [코드](https://github.com/facebook/draft-js/blob/master/src/component/utils/KeyBindingUtil.js#L38)를 보자. 해당 모듈의 지역 변수 isOSX: boolean를 가지고 hasCommandModifier 메소드는 메타 키를 누르고 있는지, 컨트롤 키를 누르고 있는지에 대한 정보를 반환한다.

## isOSX?

```javascript
const UserAgent = require("UserAgent");
//...
const isOSX = UserAgent.isPlatform("Mac OS X");
```

UserAgent 모듈은 어디에서 왔을까? 이것은 다시 페이스북이 자신들의 스크립트 파일을 패키징한 [fbjs](https://github.com/facebook/fbjs)에서 찾을 수 있는데, [UserAgent](https://github.com/facebook/fbjs/blob/master/packages/fbjs/src/useragent/UserAgent.js)는 정적 메소드 [isPlatform](https://github.com/facebook/fbjs/blob/master/packages/fbjs/src/useragent/UserAgent.js#L230)에서 [UserAgentData](https://github.com/facebook/fbjs/blob/master/packages/fbjs/src/__forks__/UserAgentData.js)를 사용하고, 드디어 우리는 최종 목적지 [UAParser.js](https://github.com/faisalman/ua-parser-js)에 다다른다.

## UAParser.js

이 라이브러리가 `window.navigator.userAgent`를 통해 `Mac OS` 여부를 구분하는 부분은 코드 [756번째 줄](https://github.com/faisalman/ua-parser-js/blob/master/src/ua-parser.js#L756)에서 찾아볼 수 있다, 있는데, 우리는 클라이언트 앱이 구동되는 운영체제가 macOS인지 아닌지만이 중요하니까 UAParser.js를 그대로 사용하지 않아도 되지 않을까? (라이브러리는 전혀 모듈화가 되어있지 않고, 매개변수 레이아웃이나 자료형도 기술되어 있지 않다.)

## isOSX(): boolean

```typescript
function isOSX(): boolean {
  const {userAgent} = window.navigator;
  return [
    /(mac\sos\sx)\s?([\w\s\.]*)/i,
    /(macintosh|mac(?=_powerpc)\s)/i,
  ].some((pattern) => pattern.test(userAgent));
}
```

해당 패턴만 발췌하였습니다, 버전 정보는 파싱하지 않습니다.
