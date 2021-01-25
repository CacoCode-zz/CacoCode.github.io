
> 生活、工作和兴趣

# To Do List

## 前端

### HTML
- [x] HTML的基本结构
- [x] 基本的表单标签
- [x] HTML5标签
- [ ] Canvas
- [ ] Websocket
- [ ] SVG


### CSS

- [x] CSS基础知识
- [x] 伪类
- [x] 水平垂直居中
- [ ] 经典布局
- [ ] BFC


### JavaScript

- [x] JavaScript基础知识
    - [x] 声明
    - [x] 内置类型
    - [x] typeof 的使用
    - [x] 数据类型转换
    - [x] 转义字符
    - [x] 运算符
- [x] 调用堆栈
    - [x] [执行上下文](https://juejin.im/post/5ba32171f265da0ab719a6d7)
        - 全局执行上下文
        - 函数执行上下文
        - eval执行上下文
    - [x] 执行上下文栈和变量对象
        - [x] 变量提升
        - [x] 函数提升
    - [x] [内存空间](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)
    - [x] [内存机制](https://juejin.im/post/5d870e6651882517a0319aa8)
        - [x] 分配内存
        - [x] 使用内存(读写)
        - [x] 释放内存
            - [x] 垃圾回收
                - [x] 引用计数(现代浏览器不再使用)
                - [x] 标记清除(常用)
    - [x] [常见内存泄露及如何避免](https://juejin.im/post/5e10a97f5188253a765ec7e1)
        - 对于不再用到的内存，没有及时释放，就叫做内存泄漏（memory leak）
        - [x] 四种常见的内存泄露
            - 意外的全局变量
                - 未声明的变量
                - 使用this创建的变量
            - 被遗忘的计时器或回调函数
                - 定时器引起
                - 观察者addEventListener
            - 脱离DOM的引用
            - 闭包
- [x] [作用域和闭包](https://zhuanlan.zhihu.com/p/35582495)
    - 划分作用域的意义
        - 函数作用域
            - 规避同名变量之间的冲突
            - 命名空间
       - 块级作用域
           - **let**可以将变量绑定到所在的任意作用域中(通常是{...}内部)
- [x] this
- [x] [原型和原型链](https://juejin.im/post/5a94c0de5188257a8929d837)
- [x] [深浅拷贝](https://juejin.im/post/5b5affaef265da0f86543bc9)
    - **浅拷贝**（shallow copy）：只复制指向某个对象的指针，而不复制对象本身，新旧对象共享一块内存；
    - **深拷贝**（deep copy）：复制并创建一个一摸一样的对象，不共享内存，修改新对象，旧对象保持不变。
- [x] 深入数组
    - [x] 数组扁平化的六种实现方式
        - 递归实现
        - reduce实现
        - flat
        - 扩展运算符
        - toString
        - apply
- [ ] 面向对象三要素
    - [ ] 封装
    - [ ] 继承
    - [ ] 多态
- [ ] [手写api](https://juejin.im/post/6844903924520992782)
- [ ] V8引擎的工作原理
    - [ ] 垃圾回收机制
    - [ ] Event loop
- [ ] 异步
    - [ ] 回调函数
    - [ ] Promise
    - [ ] generator
    - [ ] async/await
- [ ] 函数式编程
    - [ ] 偏应用
    - [ ] 函数柯里化
    - [ ] 组合函数
- [ ] ES6+


### node

- [ ] 基础知识


### TypeScript

- [ ] 基础知识


### 正则

- [ ] 正则基础知识


### 算法

- [ ] LeetCode


### 数据结构

- [ ] 二叉树
- [ ] 链表
- [ ] 数组


### 计算机网络

- [ ] 网络基础知识
- [ ] HTTP
- [ ] HTTPS
- [ ] TCP/IP
- [ ] 网络安全


### 浏览器工作原理

- [ ] 浏览器缓存
    - [ ] 缓存类型
    - [ ] 缓存位置
    - [ ] 缓存过程分析
- [ ] HTTP请求流程
- [ ] 导航流程
- [ ] 渲染流程
- [ ] 缓存机制


### 前端工程化

- [ ] Webpack
    - [ ] 基础知识
    - [ ] 构建方式
    - [ ] 配置
    - [ ] 优化
    - [ ] babel
    - [ ] loader
    - [ ] library
    - [ ] vueLoader


### 框架/库

- [ ] Vue
    - [ ] Vue-Router
    - [ ] Vuex
    - [ ] Vue源码相关


### 多端开发

- [ ] Taro
