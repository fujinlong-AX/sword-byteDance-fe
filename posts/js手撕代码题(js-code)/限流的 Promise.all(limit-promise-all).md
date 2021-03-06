## 题目
写出这段代码的输出结果：
```js
function createTask(ms) {
   return () => {
     console.log('start', ms);
     return new Promise(r => setTimeout(() => {
       console.log('end', ms);
       r(ms);
     }, ms));
   }
}
const tasks = Array(5).fill(null).map((_, i) => createTask(i * 1000));
Promise.all(tasks.map(task => task())).then(console.log);
```

接上一题，实现限制同时运行任务数的函数limitRunTask

```js
// 表示同时只能有2个任务运行，且then中得到的数组顺序和通过Promise.all得到的一致
limitRunTask(tasks, 2).then(console.log);
```

## 题解

```js
function createTask(t) {
  return () => new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(t)
    }, t);
  })
}
function limitRunTask(tasks, limit) {
  return new Promise((resolve, reject) => {
    // 当前的 promise 的 index
    let index = 0;
    // 正在运行的 promise
    let alive = 0;
    let finish = 0;
    let result = [];
    function trigger() {
      if (finish >= tasks.length) {
        resolve(result);
        return;
      }
      while (alive < limit  && index < tasks.length) {
        alive ++;
        const promise = tasks[index]();
        const curIndex = index;
        promise.then(value => {
          alive --;
          finish ++;
          result[curIndex] = value;
          trigger();
        });
        index ++;
      }
    }
    trigger();
  });
}
limitRunTask([createTask(1000), createTask(1000), createTask(1000)], 2).then(r => console.log(r))
```

## 出处

作者：NightCatS
链接：https://www.nowcoder.com/discuss/353684?type=2&order=0&pos=9&page=2
来源：牛客网


## 题目
```js
const timeout = ms => new Promise((resolve, reject) => {
  setTimeout(() => resolve(), ms);
});
const ajax1 = () => timeout(2000).then(() => {
  console.log('1');
  return 1;
});
const ajax2 = () => timeout(1000).then(() => {
  console.log('2');
  return 2;
});
const ajax3 = () => timeout(2000).then(() => {
  console.log('3');
  return 3;
});
const mergePromise = (ajaxArray) => {
};
// 1
mergePromise([ajax1, ajax2, ajax3])
  .then((data) => {
    console.log('done');
    console.log(data);
  });

// // 实现 mergePromise
// 最后输出: 1, 2, 3 'done' [1, 2, 3]
```

## 题解
```js
const mergePromise = (ajaxArray) => {
  return new Promise((resolve, reject) => {
    let i = 0, count = 0, len = ajaxArray.length;
    let res = [];
    function requestWork() {
      let index = i;
      let work = ajaxArray[i];
      if (!work) {
        resolve(res)
        return;
      };
      work().then(r => {
        i++;
        res[index] = r;
        requestWork();
      })
    }
    requestWork();
  })
};
```

## 出处


作者：老蒋

