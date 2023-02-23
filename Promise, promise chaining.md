promise
 

let promise = new Promise(function(resolve, reject) {
  // executor (제작 코드, '가수')
});
new Promise에 전달되는 함수는 executor(실행 함수)라고 부른다. executor는 new Promise가 만들어질 때 

자동으로 실행되는데 결과를 최종적으로 만들어내는 제작 코드를 포함한다.

 

executor의 인수 resolve와 reject는 자바스크립트에서 자체 제공하는 콜백이다. 개발자는 resolve와 reject를 신경 쓰지 않고 executor 안 코드만 작성하면 된다.

 

대신 executor에선 결과를 즉시 얻든 늦게 얻든 상관없이 상황에 따라 인수로 넘겨준 콜백 중 하나를 반드시 호출해야 한다.

resolve(value) — 일이 성공적으로 끝난 경우 그 결과를 나타내는 value와 함께 호출
reject(error) — 에러 발생 시 에러 객체를 나타내는 error와 함께 호출
요약하면 다음과 같습니다. executor는 자동으로 실행되는데 여기서 원하는 일이 처리됩니다. 처리가 끝나면 executor는 처리 성공 여부에 따라 resolve나 reject를 호출합니다.

한편, new Promise 생성자가 반환하는 promise 객체는 다음과 같은 내부 프로퍼티를 갖습니다.

state — 처음엔 "pending"(보류)이었다 resolve가 호출되면 "fulfilled", reject가 호출되면 "rejected"로 변합니다.
result — 처음엔 undefined이었다 resolve(value)가 호출되면 value로, reject(error)가 호출되면 error로 변합니다.
소비자 then, catch, finally

프라미스 객체는 execytor와 결과나 에러를 받을 소비 함수를 이어주는 역할을 한다.

 

than

.then은 프라미스에서 가장 중요하고 기본이 되는 메서드다.

promise.then(
  function(result) { /* 결과(result)를 다룹니다 */ },
  function(error) { /* 에러(error)를 다룹니다 */ }
);
첫번째 then의 인수는 프라미스가 이행되었을 때 실행되는 함수고 실행결과를 받는다

두번째 인수는 프라미스가 거부되었을 때 실행되고 에러를 받는다.

 

let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("완료!"), 1000);
});

// resolve 함수는 .then의 첫 번째 함수(인수)를 실행한다.
promise.then(
  result => alert(result), // 1초 후 "완료!"를 출력
  error => alert(error) // 실행되지 않음
);

위는 성공적으로 실행됐을때의 반응이다
프라미스가 거부된 경우에는 아래와 같이 두번째 함수가 실행된다.

let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("에러 발생!")), 1000);
});

// reject 함수는 .then의 두 번째 함수를 실행합니다.
promise.then(
  result => alert(result), // 실행되지 않음
  error => alert(error) // 1초 후 "Error: 에러 발생!"을 출력
);

만약 성공적으로 처리된 경우만 다루고 싶다면 .then 에 인수를 하나만 전달하면된다.

let promise = new Promise(resolve => {
  setTimeout(() => resolve("완료!"), 1000);
});

promise.then(alert); // 1초 뒤 "완료!" 출력
 

catch

에러가 발생한 경우만 다르고 싶다면 .then(null, errorHandlingFunction)같이 null을 첫번째 인수로 전달하면 된다.

.catch(errorHandlingFunction)를 써도 되는데 .catch 는 .then에 null을 전달하는 것과 동일하게 작동한다.

let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("에러 발생!")), 1000);
});

// .catch(f)는 promise.then(null, f)과 동일하게 작동합니다
promise.catch(alert); // 1초 뒤 "Error: 에러 발생!" 출력
.catch(f)는 .then(null.f)보다 간결하지만 같은 값을 출력하게된다.

 

finally

try{...}, catch{...}에 finally절이 있는 것처럼 프라미스에도 있다.

프라미스가 처리되면(이행, 거부) f가 항상 실행된다는 점에서

.finally(f) 호출은 .then(f, f)과 유사하다.

쓸모가 없어진 로딩 인디케이터(loading indicator)를 멈추는 경우같이

결과가 어떻든 마무리가 필요하면 finally가 유용하다.

 

new Promise((resolve, reject) => {
  /* 시간이 걸리는 어떤 일을 수행하고, 그 후 resolve, reject를 호출함 */
})
  // 성공·실패 여부와 상관없이 프라미스가 처리되면 실행됨
  .finally(() => 로딩 인디케이터 중지)
  .then(result => result와 err 보여줌 => error 보여줌)
 

프라미스 체이닝
프라미스 체이닝은 result가 .then 핸들러의 체인을 통해 전달된다는 점에서 착안한 아이디어다.

new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});

1초 후 최초 프라미스가 이행됩니다. – (*)
이후 첫번째 .then 핸들러가 호출됩니다. –(**)
2에서 반환한 값은 다음 .then 핸들러에 전달됩니다. – (***)
이런 과정이 계속 이어집니다.
초보자는 프라미스 하나에 .then을 여러 개 추가한 후, 이를 체이닝이라고 착각하는 경우가 있습니다. 하지만 이는 체이닝이 아닙니다.

let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});
예시의 프라미스는 하나인데 여기에 등록된 핸들러는 여러 개 입니다. 이 핸들러들은 result를 순차적으로 전달하지 않고 독립적으로 처리합니다.

 

프로미스의 resolve(1)을 순차적으로 내려간게 아니라 각각 1을 부여해 준것이기 때문에

1만 출력되게된다.

 

프라미스 반환

.then(handler)에 사용된 핸들러가 프라미스를 생성하거나 반환하는 경우가 있다.

이 경우엔 이어지는 핸들러는 프라미스가 처리될 때까지 기다리다가 처리가 완료되면 결과를 받게된다.

 

new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result); // 1

  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) { // (**)

  alert(result); // 2

  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {

  alert(result); // 4

});

예시에서 첫 번째 .then은 1을 출력하고 new Promise(…)를 반환((*))합니다.
1초 후 이 프라미스가 이행되고 그 결과(resolve의 인수인 result * 2)는 
두 번째 .then으로 전달됩니다. 두 번째 핸들러((**))는 2를 출력하고 동일한 과정이 반복됩니다.

따라서 얼럿 창엔 이전 예시와 동일하게 1, 2, 4가 차례대로 출력됩니다. 
다만 얼럿 창 사이에 1초의 딜레이가 생깁니다.

이렇게 핸들러 안에서 프라미스를 반환하는 것도 비동기 작업 체이닝을 가능하게 해줍니다.
 

leadScript 개선하기

loadScript(스크립트를 순차적으로 불러줌)

loadScript("/article/promise-chaining/one.js")
  .then(function(script) {
    return loadScript("/article/promise-chaining/two.js");
  })
  .then(function(script) {
    return loadScript("/article/promise-chaining/three.js");
  })
  .then(function(script) {
    // 불러온 스크립트 안에 정의된 함수를 호출해
    // 실제로 스크립트들이 정상적으로 로드되었는지 확인합니다.
    one();
    two();
    three();
  });
화살표 함수를 이용해 코드를 줄여준다.

loadScript("/article/promise-chaining/one.js")
  .then(script => loadScript("/article/promise-chaining/two.js"))
  .then(script => loadScript("/article/promise-chaining/three.js"))
  .then(script => {
    // 스크립트를 정상적으로 불러왔기 때문에 스크립트 내의 함수를 호출할 수 있습니다.
    one();
    two();
    three();
  });
loadScript를 호출할 때마다 프라미스를 반환되고 난 다음 .then은 이 프라미스가 이행되었을때 실행된다.

이후에 다음 스크립트를 로딩하기 위한 초기화가 진행된다.

코드가 오른쪽이 아닌 아래로 길어지기 때문에 콜백지옥엔 빠지지 않게된다.

 

요약
.then 또는 .catch , .finally의 핸들러가 프라미스를 반환하면 나머지 체인은 프라미스가

처리될 때까지 대기한다. 처리가 완료되면 프람미스의 result(값이나 에러)가 다음 체인으로 전달된다.

 

