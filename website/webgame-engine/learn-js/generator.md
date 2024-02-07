# Generators 生成器與協程

### 什麼是 Generators?

Generators (生成器) 是 ES6 引入的另一個新功能
Generators 像是一個可以隨時暫停 (pause) 和繼續 (resume) 執行的特殊函數，其實它本身也是一個 Iterator 物件

!!! tip

      一般的函數開始執行後，一定會執行到該函數結束才會離開函數，繼續執行其他的程式
      但是有的時候我們希望中斷函式的執行，並保留其狀態，這時候就會需要 Generators 派上用場。

### Generators 用法

1. 在 function 和其函式名稱中間加一個星號(\*)
2. 在函式中可以使用 yield 關鍵字，函式執行到 yield 就會中斷，並且把 yield 後面的表達式包成物件當作回傳 value 的值
3. 想要繼續執行函式則呼叫 next()，就會從上次函式中斷的地方，也就是 yield，繼續往下執行。
4. 函式執行到最後遇到 return，會將 return 後面的表達式當作 value 的回傳值，若結尾沒有 return，則最後一個回傳值 value 為 undefined。

!!! info

     呼叫next()回傳的物件為{value : any,done : bool}
     value是函式內部執行到yield關鍵字後面的表達式
     done為布林值，若函式執行完畢done值為true，否則為fales。

```ts
//generator
function* attack() {
  for (let i = 0; i < 4; i++) {
    yield `階段${i}`; //執行到這行中斷函式，並回傳 `階段${i}`
  }
  return "end";
}

let gen = attack(); //gen為generator物件
let a = gen.next(); //呼叫next()取得函式中斷時的回傳值和執行狀態
console.log(a); //{value: '階段0', done: false}

let b = gen.next();
console.log(b); //{value: '階段1', done: false}

let c = gen.next();
console.log(c); //{value: '階段2', done: false}

let d = gen.next();
console.log(d); //{value: '階段3', done: false}

let e = gen.next();
console.log(e); //{value: 'end', done: true}

let f = gen.next();
console.log(f); //{value: undefined, done: true}
```

Generator 簡單的概念就是把 1 個函式拆分成多個部分分次執行，
在分次執行的情況下，有時候也需要傳遞值到函式內，
而 generator 的 next()也可以傳遞一個參數，這個參數會在函式內部取代上一次執行到 yield 關鍵字的值。

```ts
//generator
function* attack() {
  let i = 0;
  //模擬攻擊所花的延遲時間
  while (i < 4) {
    let x = yield `階段${i}`; //執行到這行中斷函式，並回傳i
    console.log(x);
    i += 1;
  }
}

let gen = attack(); //gen為generator物件
let obj = gen.next(100); //這個參數100是沒用的，見下方info
console.log(obj);
let j = 0;
while (!obj.done) {
  obj = gen.next(j); //用j取代函式內部的 yield `階段${i}`
  console.log(obj);
  j += 5;
}

//{value: '階段0', done: false}
// 0
//{value: '階段1', done: false}
// 5
//{value: '階段2', done: false}
// 10
//{value: '階段3', done: false}
// 15
//{value: undefined, done: true}
```

!!! Warning

    第1次呼叫next()傳遞的參數沒有做用，因為next()的參數是取代上一次函式執行到yield的回傳值，而第一次呼叫next()，並沒有上一次的yield可以取代，因此第一次呼叫next()傳遞參數沒有意義。

#### yield\* 用法

yield\* 這關鍵字可以用來在一個 generator 裡面執行另一個的 generator。

```ts
function* g1() {
  yield 2;
  yield 3;
  yield 4;
}

function* g2() {
  yield 1;
  yield* g1();
  yield 5;
}

var iterator = g2();

for (let v of g2()) {
  console.log(v);
}
// 依序會輸出 1 2 3 4 5
```

### generator 實現概念-協程

#### 什麼是協程?

協程（Coroutine）與傳統的線程（Thread）或進程（Process）不同，協程處在線程的環境中，一個線程可以存在多個協程，可以將協程理解為線程中的一個個小任務，協程的控制權是由開發者的程式碼來控制。協程可以暫停執行、保存當前狀態，並在需要時恢復執行。這使得協程能夠有效地處理非同步的任務，並且能夠更靈活地管理程序的執行流程。

#### 協程的運作

由於 js 是單線程執行的，而一個線程一次只能執行一個協程，所以在處理非同步的任務時，並不是同時執行多個協程，而是靠中斷的方式來達到非同步的效果，例如:A 協程執行到一半，想要執行 B 協程，就要先暫停 A 協程將 js 線程的控制權轉交給 B 協程，這時候就會執行 B 協程，A 協程就處於暫停的狀態。

!!! info

    協程的好處是不受作業系統控制，可以由開發者的程式碼自行控制，也不像Process和Thread需要較多的資源進行切換，因此協程的切換通常更加高效和省資源。

### 使用 generator 範例

##### 生成無窮序列:

generator 可以在需要的時候生成數字而不用事先將整個數列儲存在陣列中

```ts
function* naturalNumbers() {
  let number = 1;
  while (true) {
    yield number++;
  }
}

let generator = naturalNumbers();
console.log(generator.next().value); // 輸出: 1
console.log(generator.next().value); // 輸出: 2
console.log(generator.next().value); // 輸出: 3
// 繼續可以一直遞增至無窮
```

**使用 Generator Function 建立 Iterator :**

for...of 可以自動疊代 Generator 函數時生成的 Iterator 物件，且此時不再需要呼叫 next 方法。

```ts
function* str() {
  yield "Hello";
  yield "World";
  yield "!!!";
  return "???";
}

for (let n of str()) {
  console.log(n); // Hello World !!!
}
```

一般遍歷的方法像 for...of，只要遇到 done=true 就會結束立刻結束遍歷，所以上面範例的輸出結果不會有"???"

##### 非同步的操作管理:

當需要按照順序執行非同步任務時，可以待配 Promise 使用，使 generator 函數先暫停，等待非同步任務完成後再繼續執行。

```ts
function* asyncTaskGenerator() {
  let result1 = yield asyncTask1();
  console.log(result1);
  let result2 = yield asyncTask2();
  console.log(result2);
}

function asyncTask1() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("Async Task 1 Done"), 1000);
  });
}

function asyncTask2() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("Async Task 2 Done"), 1000);
  });
}

let generator = asyncTaskGenerator();
generator.next().value.then((result1) => {
  generator.next(result1).value.then((result2) => {
    generator.next(result2);
  });
});

//延遲1秒
//Async Task 1 Done
//延遲1秒
//Async Task 2 Done
```
