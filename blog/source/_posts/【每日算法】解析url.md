---
title: 【每日算法】解析url
date: 2021-11-03 16:58:18
tags:
    - JavaScript
    - 算法
categories:
    - 前端算法
keywords: "面试,js算法"
description: 与前端相关的算法
cover: https://img.cdn.sugarat.top/mdImg/MTY0NzY3OTY4OTgzMA==647679689830


---

## 解析url

```js
function urlSearch(href) {
    let name,value;
    let str = href
    // 取出问好后面的字符串
    let num = str.indexOf('?')
    str = str.substr(num+1)

    // 以&分成数组元素
    let arr = str.split("&")
    let json = {}
    for(let i = 0;i < arr.length;i++){
        num = arr[i].indexOf("=")
        if(num > 0){

            // 以等号将key value分别取出然后加入对象
            name = arr[i].substring(0,num)
            value = arr[i].substr(num+1)
            json[name] = value
        }
    }
    return json
}

let s = urlSearch('https://www.shixiseng.com/interns?page=1&type=company&keyword=oppo&area=&months=&days=&degree=&official=&enterprise=&salary=-0&publishTime=&sortType=&city=%E5%85%A8%E5%9B%BD&internExtend=')
console.log(s)

```

