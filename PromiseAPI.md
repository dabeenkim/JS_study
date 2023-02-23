프라미스엔 5가지 정적 메서드가 있다.

 

Primise.all

 

let promise = Promise.all([...promises...]);
요소 전체가 프라미스인 배열(엄밀히 따지면 이터러블객체지만 대부분 배열이다)을 받고 새로운 프라미스 반환

배열 안 프라미스가 모두 처리되면 새로운 프라미스가 이행됨. 배열 안 프라미스의 결괏값을 담은

배열이 새로운 프라미스의 result가 된다.

 

Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert); // 프라미스 전체가 처리되면 1, 2, 3이 반환된다. 각 프라미스는 배열을 구성하는 요소가 된다.
 

Promise.allSettled
.Promise.all은 프라미스가 하나라도 거절되면 전체를 거절한다.

프라미스값이 모두 필요할때 사용할 수 있다.

 

Promise.all([
  fetch('/template.html'),
  fetch('/style.css'),
  fetch('/data.json')
]).then(render); // render 메서드는 fetch 결과 전부가 있어야 제대로 동작한다
Promise.allSettled는 모든 프라미스가 처리될 때까지 기다린다. 반환되면 이렇게 나오게된다.

응답이 성공할 경우 – {status:"fulfilled", value:result}
에러가 발생한 경우 – {status:"rejected", reason:error}
fetch를 사용해 여러 사람의 정보를 가져올때 여러 요청중 하나가 실패해도 다른 요청 결과는

여전히 필요하다 이럴때 promise.allSettled가 사용될 수 있다.

 

let urls = [
  'https://api.github.com/users/iliakan',
  'https://api.github.com/users/Violet-Bora-Lee',
  'https://no-such-url'
];

Promise.allSettled(urls.map(url => fetch(url)))
  .then(results => { // (*)
    results.forEach((result, num) => {
      if (result.status == "fulfilled") {
        alert(`${urls[num]}: ${result.value.status}`);
      }
      if (result.status == "rejected") {
        alert(`${urls[num]}: ${result.reason}`);
      }
    });
  });
  
(*)로 표시한 줄의 results는 다음과 같을 겁니다.


[
  {status: 'fulfilled', value: ...응답...},
  {status: 'fulfilled', value: ...응답...},
  {status: 'rejected', reason: ...에러 객체...}
]

Promise.allSettled를 사용하면 이처럼 각 프라미스의 상태와 값 또는 에러를 받을 수 있습니다.
 

폴리필
브라우저가 Promise.allSettled를 지원하지 않는다면 폴리필을 구현하면된다.

if(!Promise.allSettled) {
  Promise.allSettled = function(promises) {
    return Promise.all(promises.map(p => Promise.resolve(p).then(value => ({
      status: 'fulfilled',
      value
    }), reason => ({
      status: 'rejected',
      reason
    }))));
  };
}
여기서 promise.map은 입력값을 받아 p => promise.resolve(p)로 입력밧을 프라미스로 변화시킨다.(프라미스가 아닌 값을 받은경우) 그리고 모든 프라미스에 .then핸들러가 추가된다.

 

then핸들러는 성공한 프라미스의 결괏값 value를 {status : 'fulfilled', value}로 실패한 프라미스의 결괏값 reason을

{status : 'rejected', reason}으로 변경한다. Promise.allSettled의 구성과 동일하다.

 

Promise.race
let promise = Promise.race(iterable);
 

Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("에러 발생!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
첫번째 프라미스가 가장 빨리 처리된상태이기 때문에 첫번째 프라미스의 결과가 result값이 된다.

race의 승자가 나타난 순간 다른 프라미스의 결과 또는 에러가 무시된다.

 

Promise.resolve
결괏값이 value인 이행 상태 프라미스를 생성한다.

let promise = new Promise(resolve => resolve(value));
loadCached는 인수로 받은 URL을 대상으로 fetch를 호출하고 그 결과를 기억(cache)한다. 나중에

동일한 URL을 대상으로 fatch를 호출하면 캐시에서 호출 결과를 즉시 가져오는데

이때 Promise.resolve를 사용해 캐시 된 내용을 프라미스로 만들어 반환 값이 항상 프라미스 되게한다.

 

let cache = new Map();

function loadCached(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url)); // (*)
  }

  return fetch(url)
    .then(response => response.text())
    .then(text => {
      cache.set(url,text);
      return text;
    });
}
loadCached를 호출하면 프라미스가 반환된다는 것이 보장되기 때문에 loadCached(url), then( ... )을 사용할 수

있다. loadCached 뒤에 언제나 .then을 쓸 수 있게된다.

 

Promise.reject
Promise.reject(error)는 결괏값이 error인 거부 상태 프라미스를 생성한다.

실무에선 쓸일이 거의 없다.

let promise = new Promise((resolve, reject) => reject(error));
 

요약

 

Promise 클래스에는 5가지 정적 메서드가 있습니다.

Promise.all(promises) – 모든 프라미스가 이행될 때까지 기다렸다가 그 결괏값을 담은 배열을 반환합니다. 주어진 프라미스 중 하나라도 실패하면 Promise.all는 거부되고, 나머지 프라미스의 결과는 무시됩니다.
Promise.allSettled(promises) – 최근에 추가된 메서드로 모든 프라미스가 처리될 때까지 기다렸다가 그 결과(객체)를 담은 배열을 반환합니다. 객체엔 다음과 같은 정보가 담깁니다.
status: "fulfilled" 또는 "rejected"
value(프라미스가 성공한 경우) 또는 reason(프라미스가 실패한 경우)
Promise.race(promises) – 가장 먼저 처리된 프라미스의 결과 또는 에러를 담은 프라미스를 반환합니다.
Promise.resolve(value) – 주어진 값을 사용해 이행 상태의 프라미스를 만듭니다.
Promise.reject(error) – 주어진 에러를 사용해 거부 상태의 프라미스를 만듭니다.
실무에선 다섯 메서드 중 Promise.all을 가장 많이 사용합니다.

프라미스화
콜백 받는 함수를 프라미스로 반환하는 함수로 바꾸는 것을 프라미스화(promisification)이라고 한다.

기능을 구현하다 보면 콜백보다는 프로미스가 더 편리하기 때문에 콜백 기반 함수와 라이브러리를

프로미스를 반환하는 함수로 바꾸는게 좋은 경우가 생기게된다.

 

function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`${src}를 불러오는 도중에 에러가 발생함`));

  document.head.append(script);
}

// 사용법:
// loadScript('path/script.js', (err, script) => {...})
loadScript(src, callback)를 이제 프라미스화 해보면  새로운 함수 loadScriptPromise(src)는 

loadScript와 동일하게 동작하지만 callback을 제외한 src만 인수로 받아야하고 프로미스를 반환해야한다.

 

let loadScriptPromise = function(src) {
  return new Promise((resolve, reject) => {
    loadScript(src, (err, script) => {
      if (err) reject(err)
      else resolve(script);
    });
  })
}

// 사용법:
// loadScriptPromise('path/script.js').then(...)
loadScriptPromise는 기존 함수 loadScript에 모든 일을 위임한다.

loadScript의 콜백은 스크립트 로딩 상태에 따라 이행 혹은 거부 상태의 프로미스를 반환한다.

 

하지만 실무에선 하나의 함수가 아닌 여러 개의 함수를 프로미스화 해야하기 때문에 

헬퍼함수를 만드는게 좋다. 프로미스를 적용할 함수 f를 받고 래퍼 함수를 반환하는 함수 promisify(f)를 만든다.

 

function promisify(f) {
  return function (...args) { // 래퍼 함수를 반환함
    return new Promise((resolve, reject) => {
      function callback(err, result) { // f에 사용할 커스텀 콜백
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      }

      args.push(callback); // 위에서 만든 커스텀 콜백을 함수 f의 인수 끝에 추가합니다.

      f.call(this, ...args); // 기존 함수를 호출합니다.
    });
  };
};

// 사용법:
let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
좀더 진화한 함수도 만들수 있다.

promisify(f, true) 형태로 호출하게 되면 프라미스결과는 콜백의 성공 케이스(results)를 담은 배열

[res1, res2, ...]이 된다.

 

// 콜백의 성공 결과를 담은 배열을 얻게 해주는 promisify(f, true)
function promisify(f, manyArgs = false) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function callback(err, ...results) { // f에 사용할 커스텀 콜백
        if (err) {
          reject(err);
        } else {
          // manyArgs가 구체적으로 명시되었다면, 콜백의 성공 케이스와 함께 이행 상태가 됩니다.
          resolve(manyArgs ? results : results[0]);
        }
      }

      args.push(callback);

      f.call(this, ...args);
    });
  };
};

// 사용법:
f = promisify(f, true);
f(...).then(arrayOfResults => ..., err => ...)
 

주의할 점은

콜백화를 완전히 대체하지 못하며 

프라미스는 하나의 결과만 가질 수 있지만 콜백은 여러번 호출할 수 있기때문이다.

프라미스는 콜백을 단 한번만 호출하는 함수에만 적용하는게 좋다.

프라미스화한 함수의 콜백을 여러번 호출해도 두번째부터는 무시된다.

