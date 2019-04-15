---
layout: post
title: 魂斗罗中的“防抖”和“节流”
date: 2019-04-15 18:47:05
description: 防抖和节流有何区别
tags: [javascript]
---

“防抖”和“节流”本质上都是避免高频函数的多次调用，大家小时候肯定玩过一款魂斗罗的游戏，其中子弹各有特点，在不考虑子弹威力，只看射击频率，假设每按下一次子弹键，触发一次发射事件。

***L型：需要在按下子弹键后，松开按键一段时间，子弹才能发射出去***

![]({{site.baseurl}}/assets/img/hdl_l.gif)

这就涉及到了“防抖”的概念，“防抖”指的是`n`秒内函数只被调用一次，且每次的调用都重新计算时间，实现起来也很简单：

```javascript
function debounce (func, delay) {
    let timeoutId
    const debounced = (...params) => {
        clearTimeout(timeoutId)
        timeoutId = setTimeout(() => {
            func(...params)
        }, delay)
    }

    return debounced
}
```

```javascript
function shoot (i) {
  console.log(Date.now(), 'L子弹发射被执行',i)
  // ... 射击的逻辑
} 

const debounced = debounce(shoot, 1000) // L型子弹只有松开按键1秒后才能发射出去

for (let i = 0; i < 5; i++) {
  // 假设你以每秒5下的频率按下发射键
  setTimeout(() => {
    console.log(Date.now(), 'L发射事件', i)
    debounced(i)
  }, i * 200)
}
```


![]({{site.baseurl}}/assets/img/debounce.png)

你快速按下5次发射键，只窜出去一根小火苗，“防抖”的重点是在触发事件停止n秒后才去执行动作，在这之前的触发事件都将被抑制。








***M型：发射间隔固定500ms***

![]({{site.baseurl}}/assets/img/hdl_m.gif)

M型子弹发射间隔固定，每秒至多2发，“节流”指的是n秒内执行一次，执行一次后，只有大于执行周期，才会执行第二次



```javascript
function throttle (func, interval) {
    let timeoutId, preExecuteAt = Date.now()
    const handle = (...params) => {
        func(...params)
        preExecuteAt = Date.now()
    }
    const throttled = (...params) => {
        clearTimeout(timeoutId)
        const waited = Date.now() - preExecuteAt
        if (waited >= interval) {
            handle(...params)
        } else {
            timeoutId = setTimeout(() => {
                handle(...params)
            }, interval - waited)
        }
    }
    return throttled
}
```



```javascript
function shoot (i) {
  console.log(Date.now(), 'M子弹发射被执行',i)
  // ... 射击的逻辑
} 

const throttled = throttle(shoot, 500) // M型子弹发射间隔0.5秒，每秒最多发射2发

for (let i = 0; i < 5; i++) {
  // 假设你以每秒5下的频率按下发射键
  setTimeout(() => {
    console.log(Date.now(), 'M发射事件', i)
    throttled(i)
  }, i * 200)
}
```





![]({{site.baseurl}}/assets/img/throttle.png)

throttle 会强制函数以固定速率执行，每次间隔500ms以上



***"防抖"和"节流"有着各自的应用场景***

- 防抖：文本输入做表单验证(异步ajax，做一次验证就好)
- 节流：动画渲染