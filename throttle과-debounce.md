# throttle과 debounce

scroll 이벤트나 resize 이벤트는 브라우저의 렌더링 타이밍보다 더 잦게 발생하기 때문에, 그 이벤트가 발생하는 족족 DOM 조작 등의 비싼 작업을 수행하는 것은 좋지 않다. 대신 throttle, 함수의 호출을 특정 시간동안 한 번으로 제한, 혹은 debounce, 함수의 호출을 특정 시간만큼 지연시켜 가장 마지막 전달된 인자 목록으로 호출, 등등의 비용 절감의 기술이 필요하다.

## lodash의 throttle, debounce

lodash의 [throtte](https://github.com/lodash/lodash/blob/master/throttle.js)을 알아보자. 이걸 살펴보면 세번째 인자인 옵션의 leading, trailing, maxWait 속성을 작성해서 던지는 debounce 메소드의 특별한 버전이다. leading과 trailing은 기본값 true를 지정 후, 런타임에 호출한 쪽의 옵션을 재할당하고 있으며 maxWait는 두번째 인자 wait 값을 고정하여 호출.

[debounce](https://github.com/lodash/lodash/blob/master/debounce.js)는 매개변수가 셋 있다. wait 인자가 nullish하다면 requestAnimationFrame을 사용한다. 최종적으로는 debounce 함수 가장 마지막에 선언된 지역 함수 debounced를 반환하는데, 이 지역 함수 안에서는 런타임에 입력된 인자에 따라 내부의 적당한 지역 함수를 조합하여 함수를 지연, 호출한다. 그 외에도 반납하는 debounced 함수에 cancel, flush, pending 같은 헬퍼 메소드 및 속성을 전달, 상황에 맞게 활용할 수 있도록 하고 있다. 다만 공식 [문서](https://lodash.com/docs#debounce)에는 그런 내용이 제대로 작성되어 있진 않네요.

## 만들어 보자, throttle

이것은 메소드의 최근 실행 시간을 기록하는 변수만 있으면 된다.

```javascript
function throttle(fn, wait) {
  let timestamp = -wait || 0;
  return function throttled(...args) {
    const now = performance.now();
    now - timestamp > wait && (timestamp = now) && fn.apply(this, args);
  };
}
```

무엇을 설명할까.

## 만들어 보자, debounce

이것은 위 throttle보다 더 적극적으로 cancel 함수를 사용한다. 지정된 wait 만큼 이벤트 핸들러를 지연시키며, 지연된 상태라면 이전 호출을 취소한다.

```javascript
function debounce(fn, wait) {
  let cancelId = null;
  const hasWait = Number.isSafeInteger(wait);
  const [timer, cancel] = hasWait
    ? [setTimeout, clearTimeout]
    : [requestAnimationFrame, cancelAnimationFrame];
  return function debounced(...args) {
    cancel(cancelId);
    cancelId = timer(fn.bind(this, ...args), wait);
  };
}
```

이것도 이것으로 되었다.
