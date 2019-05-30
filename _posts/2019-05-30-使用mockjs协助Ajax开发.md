---
layout:     post
title:      使用mockjs进行Ajax请求开发
subtitle:   手把手教你使用mockjs
date:       2019-05-30
author:     AchanYao
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 前端
    - JavaScript
---

## 前言

在进行前端开发过程中，向后端发送请求，接收并处理数据，但是后端还没做好。这时候前端工程师怎么办，总不能傻等后端做好吧。所以就有了mockjs。

## Mock.js 介绍

> Mock.js[官网](http://mockjs.com/)

### 优点

* **让前后端完全分离** *让前端工程师完全独立于后端进行开发*
* **开发无侵入** *不需要修改既有代码，就可以拦截 Ajax 请求，返回模拟的响应数据*
* **数据类型丰富** *支持生成随机的文本、数字、布尔值、日期、邮箱、链接、图片、颜色等*
* **增加单元测试真实性** *通过随机数据，模拟各种场景*
* **用法简单** *符合直觉的接口*
* **方便扩展** *支持支持扩展更多数据类型，支持自定义函数和正则*

### 安装

#### Node

```bash
# 安装
npm install mockjs
```

```bash
// 使用 Mock
var Mock = require('mockjs')
var data = Mock.mock({
    // 属性 list 的值是一个数组，其中含有 1 到 10 个元素
    'list|1-10': [{
        // 属性 id 是一个自增数，起始值为 1，每次增 1
        'id|+1': 1
    }]
})
// 输出结果
console.log(JSON.stringify(data, null, 4))
```

#### 超链接

有时候你可能没有node环境或是其他原因，需要以超链接形式导入

```html
<script src="https://yaoyuxuan.cn/static/mockjs/mock.js"></script>
```

这是我自己做的。

### 方法使用

> 看看[文档](http://mockjs.com/examples.html)就行，比我讲的细。

### 示例

假如你需要做一个图书管理系统，其中一个部分就是动态获取书籍信息（编号，书名，作者）。发送的Ajax请求使用axios框架。

这是我们的请求代码：

```javascript
function getAllBooks() {
    axios({
        method: 'POST',
        url: '/get_all_books'
    }).then(res => {
        // 在控制台上打印所获取到的Json数据
        console.log(res.data);
    })
}
```

使用Mock.js拦截该请求：

```javascript
function mockGetAllBooks()
{
    // 模拟返回的Json数据
    return Mock.mock({
        'status': 200,
        'books|2-20': [{
            'ID|1-384920': 10,
            'name': Mock.mock('@name'),
            'author|1-3': [{
                'name': Mock.mock('@name')
            }]
        }]
    });
    // 这里解释一下各个属性的含义
    // status是状态码
    // books表示现有的书目，是个长度2-20的随机数组
    // books[i].ID 书本ID
    // books[i].name 书名
    // books[i].author 是个长度1-3的随机数组，表示作者
}

// 配置Mock.js拦截，必须配置，不然Mock.js会拦截不到，控制台会报错
Mock.mock('/get_all_books', mockGetAllBooks());
// 第一个参数指要拦截的请求，第二个参数表示处理该请求的方法。
```

-----
带有参数的Mock拦截：

还是那个案例，如果我不想获取全部书目了，只想获取指定ID的书籍。

```javascript
function getBook(id) {
    axios({
        method: 'POST',
        url: '/get_book',
        data: {
            'id': id
        }
    }).then(res => {
        // 在控制台上打印所获取到的Json数据
        console.log(res.data);
    })
}

// 编写对应的Mock模拟函数
function mockGetBook(id)
{
    return Mock.mock({
        'ID': id,
        'name': Mock.mock('@name'),
        'author|1-3': [{
            'name': Mock.mock('@name')
        }]
    });
}

//配置拦截器
Mock.mock('/get_book', function (options) {
    let paras = JSON.parse(options.body);
    return mockGetBook(paras.id);
})
```

## 最后

吐槽一下学院的课程安排，今年我们JavaEE方向课有个Ajax应用开发。我个人认为这门课实在没有开设的必要，只需要在做应用的时候老师们简单向学生介绍一下，什么是Ajax，Ajax能做什么等一些基本概念就行。而不是像现在这样，从XMLHttpRequest写起，现在的Javaee开发是框架的时代，从底层写起还不如去学学数据结构，文件存储，算法一些更基础的东西来得实在。而且jQuery或是axios那些Ajax框架已经封装那么好了，jQuery这门课已经学过了，我们为什么不直接用那些呢，非得每次实验课去调试各种浏览器兼容问题，而不是专注于Ajax的效果。😑😑
