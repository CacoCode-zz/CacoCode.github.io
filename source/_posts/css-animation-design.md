---
title: 简易CSS动画设计（持续更新⏳）
date: 2020-05-26 20:44:52
tags:
- CSS
- 动画设计
categories:
- 前端
---

<link href="/scss/something-about-css.css" rel="stylesheet"></link>
<link href="/scss/css-animation-design/base.css" rel="stylesheet"></link>

> 一切来源于生活，看到或想到感兴趣的东西，仅仅用自己想要的形式表现出来。

### 文本聚焦动画

- 利用伪元素实现文本选中效果，光标移动上去查看效果

  <span class="transition-demo demo-1">zcheng.site</span>
  <details>
    <summary>显示代码</summary>
    ```html HTML
    <span class="transition-demo demo-1">zcheng.site</span>
    ```
    ```scss SCSS
    .transition-demo {
      position: relative;
      cursor: pointer;
      color: #24292e;
      transition: all 0.1s;
      /*demo1*/
      &.demo-1 {
        &::after {
          content: "";
          position: absolute;
          left: 0;
          right: 0;
          bottom: 0;
          transition: transform 0.3s ease;
          height: 2px;
          background: linear-gradient(to right, #24292e, #666);
          transform: scaleX(0);
          transform-origin: center right;
        }
        &:hover::after {
          transform: scaleX(1);
          transform-origin: center left;
        }
      }
    }
    ```
  </details>

  <span class="transition-demo demo-2">zcheng.site</span>
  <details>
    <summary>显示代码</summary>
    ```html HTML
    <span class="transition-demo demo-2">zcheng.site</span>
    ```
    ```scss SCSS
    .transition-demo {
      position: relative;
      cursor: pointer;
      color: #24292e;
      transition: all 0.1s;
      /*demo-2*/
      &.demo-2 {
        &:hover {
          color: #fff;
          z-index: 1;
        }
        &::after {
          content: "";
          position: absolute;
          top: 0;
          bottom: 0;
          left: -2px;
          right: -2px;
          transition: transform 0.1s linear;
          background: linear-gradient(#24292e, #666);
          transform: scaleY(0);
          transform-origin: bottom;
          z-index: -1;
        }
        &:hover::after {
          transform: scaleY(1);
        }
      }
    }
    ```
  </details>

  <span class="transition-demo demo-3-parent"><span class="transition-demo demo-3-child">zcheng.site</span></span>
  <details>
    <summary>显示代码</summary>
    ```html HTML
    <span class="transition-demo demo-3-parent">
      <span class="transition-demo demo-3-child">zcheng.site</span>
    </span>
    ```
    ```scss SCSS
    .transition-demo {
      position: relative;
      cursor: pointer;
      color: #24292e;
      transition: all 0.1s;
      @mixin demo-3($transform) {
        content: "";
        position: absolute;
        background: #262626;
        transform: $transform;
        transition: transform 0.3s;
      }
      &.demo-3-parent {
        padding: 2px 0;
        &::before, &::after {
          @include demo-3(scaleX(0));
          left: 0;
          right: 0;
          height: 2px;
        }
        &::before {
          top: 0;
          transform-origin: center right;
        }
        &::after {
          bottom: 0;
          transform-origin: center left;
        }
        &:hover::before {
          transform-origin: center left;
          transform: scaleX(1);
        }
        &:hover::after {
          transform-origin: center right;
          transform: scaleX(1);
        }
      }
      &.demo-3-child {
        padding: 0 2px;
        &::before, &::after {
          @include demo-3(scaleY(0));
          top: 0;
          bottom: 0;
          width: 2px;
        }
        &::before {
          left: 0;
          transform-origin: center top;
        }
        &::after {
          right: 0;
          transform-origin: center bottom;
        }
        &:hover::before {
          transform-origin: center bottom;
          transform: scaleY(1);
        }
        &:hover::after {
          transform-origin: center top;
          transform: scaleY(1);
        }
      }
    }
    ```
  </details>

### 简单的加载动画

- 利用`transform`结合伪元素实现加载动画

  <div class="animation-demo demo-1"></div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="animation-demo demo-1"></div>
    ```
    ```scss SCSS
    .animation-demo {
      &.demo-1 {
        width: 2em;
        height: 2em;
        border-radius: 50%;
        background: gray;
        animation: animation-demo-1 1s infinite ease-in-out;
        @keyframes animation-demo-1 {
          0% {
            transform: scale(0);
          }
          100% {
            opacity: 0;
            transform: scale(1);
          }
        }
      }
    }
    ```
  </details>

  <div class="animation-demo demo-2"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="animation-demo demo-2"></div>
    ```
    ```scss SCSS
    .animation-demo {
      &.demo-2 {
        @mixin demo-2-common($delay) {
          width: 1.5em;
          height: 1.5em;
          border-radius: 50%;
          animation: animation-demo-2 2s infinite ease-in-out both $delay;
        }
        position: relative;
        @include demo-2-common(0s);
        &::before {
          @include demo-2-common(0.2s);
          content: "";
          position: absolute;
          left: 2em;
        }
        &::after {
          @include demo-2-common(0.4s);
          content: "";
          position: absolute;
          left: 4em;
        }
        @keyframes animation-demo-2 {
          0%, 100% {
            box-shadow: 0 1.5em 0 -1.5em gray;
          }
          50% {
            box-shadow: 0 1.5em 0 0 gray;
          }
        }
      }
    }
    ```
  </details>

- 类支付宝APP加载动画😁

  <div class="border-demo">
    <div class="demo-5-1"></div>
  </div>
  <div class="border-demo">
    <div class="demo-5-2"></div>
  </div>
  <div class="border-demo">
    <div class="demo-5-3"></div>
  </div><br>

- 更简单的实现方式，通过`transform`倾斜即可

  <div class="border-demo demo-6"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-6"></div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      &.demo-6 {
        @mixin border-demo-6($delay) {
          height: 1.5rem;
          width: 1rem;
          background: transparent;
          border-radius: 3px;
          animation: border-demo-6 1.8s infinite ease-in-out $delay both;
        }
        @include border-demo-6(0s);
        position: relative;
        margin: 0 40px;
        transform: skewX(-25deg);
        &::before {
          @include border-demo-6(-0.16s);
          content: "";
          position: absolute;
          left: -20px;
          top: 0;
        }
        &::after {
          @include border-demo-6(0.16s);
          content: "";
          position: absolute;
          right: -20px;
          top: 0;
        }

        @keyframes border-demo-6 {
          0%, 80%, 100% {
            box-shadow: 0 1.5rem 0 -0.75rem gray;
          }
          40% {
            box-shadow: 0 1.5rem 0 0 gray;
          }
        }
      }
    }
    ```
  </details>

- 旋转效果

  <div class="rotate-demo demo-1"></div><br><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="rotate-demo demo-1"></div>
    ```
    ```scss SCSS
    .rotate-demo {
      &.demo-1 {
        display: inline-block;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        box-shadow: 0 -21px 0 0 #a3afb7, 14px -14px 0 0 #a3afb7, 21px 0 0 0 #a3afb7, 14px 14px 0 0 #a3afb7, 0 21px 0 0 #a3afb7, -14px 14px 0 0 #a3afb7, -21px 0 0 0 #a3afb7, -14px -14px 0 0 #a3afb7;
        animation: rotate-demo-1 1.5s infinite ease;
      }
      @keyframes rotate-demo-1 {
        0% {
          transform: rotate(0deg);
        }
        80% {
          transform: rotate(275deg);
        }
        100% {
          transform: rotate(270deg);
        }
      }
    }
    ```
  </details><br>

  <div class="rotate-demo demo-2"></div><br>

  <details>
    <summary>显示代码</summary>

    ```html HTML
    <div class="rotate-demo demo-2"></div>
    ```

    ```scss SCSS
    &.demo-2 {
      display: inline-block;
      height: 4.5rem;
      width: 4.5rem;
      border: 4px solid gray;
      border-bottom-color: transparent;
      border-radius: 50%;
      animation: rotate-demo-2 1s infinite linear;
    }
    @keyframes rotate-demo-2 {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }
    ```
  </details>

### 按钮效果

- 按钮边角动画

  <div class="border-button-demo demo-1">
    <button type="button">看看我</button>
  </div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-button-demo demo-1">
      <button type="button">看看我</button>
    </div>
    ```
    ```scss SCSS
    .border-button-demo {
      &.demo-1 {
        display: inline-block;
        position: relative;
        &:hover {
          &::before {
            content: "";
            z-index: 10;
            position: absolute;
            border-top-left-radius: 3px;
            border-bottom-right-radius: 3px;
            border-top-right-radius: 3px;
            animation: border-button-top-right 1s infinite ease-in-out both;
          }
          &::after {
            content: "";
            z-index: 10;
            position: absolute;
            border-top-right-radius: 3px;
            border-bottom-left-radius: 3px;
            border-bottom-right-radius: 3px;
            animation: border-button-bottom-right 1s infinite ease-in-out both;
          }
        }
        button {
          position: relative;
          padding: 1.2rem 2rem;
          line-height: 1;
          background: #fff;
          border-width: 0;
          background: #eee;
          color: #606266;
          text-align: center;
          cursor: pointer;
          border-radius: 4px;
          outline: none;
          &:hover {
            color: #409eff;
            border-color: transparent;
            background-color: #ecf5ff;
            border-width: 0;
            &::before {
              content: "";
              z-index: 10;
              position: absolute;
              border-top-right-radius: 3px;
              border-bottom-left-radius: 3px;
              border-top-left-radius: 3px;
              animation: border-button-top-left 1s infinite ease-in-out both;
            }
            &::after {
              content: "";
              z-index: 10;
              position: absolute;
              border-bottom-right-radius: 3px;
              border-top-left-radius: 3px;
              border-bottom-left-radius: 3px;
              animation: border-button-bottom-left 1s infinite ease-in-out both;
            }
            @keyframes border-button-top-left {
              0%, 100% {
                left: -.3rem;
                top: -.3rem;
                border: .6rem solid #409eff;;
                border-right-color: transparent;
                border-bottom-color: transparent;
              }
              50% {
                left: -.5rem;
                top: -.5rem;
                border: .6rem solid rgb(160, 207, 255);
                border-right-color: transparent;
                border-bottom-color: transparent;
              }
            }
            @keyframes border-button-bottom-left {
              0%, 100% {
                left: -.3rem;
                bottom: -.3rem;
                border: .6rem solid #409eff;
                border-right-color: transparent;
                border-top-color: transparent;
              }
              50% {
                left: -.5rem;
                bottom: -.5rem;
                border: .6rem solid rgb(160, 207, 255);
                border-right-color: transparent;
                border-top-color: transparent;
              }
            }
            @keyframes border-button-top-right {
              0%, 100% {
                right: -.3rem;
                top: -.3rem;
                border: .6rem solid #409eff;
                border-left-color: transparent;
                border-bottom-color: transparent;
              }
              50% {
                right: -.5rem;
                top: -.5rem;
                border: .6rem solid rgb(160, 207, 255);
                border-left-color: transparent;
                border-bottom-color: transparent;
              }
            }
            @keyframes border-button-bottom-right {
              0%, 100% {
                right: -.3rem;
                bottom: -.3rem;
                border: .6rem solid #409eff;
                border-left-color: transparent;
                border-top-color: transparent;
              }
              50% {
                right: -.5rem;
                bottom: -.5rem;
                border: .6rem solid rgb(160, 207, 255);
                border-left-color: transparent;
                border-top-color: transparent;
              }
            }
          }
        }
      }
    }
    ```
  </details>

### 类似沙漏加载动画

- 沙漏

  <div class="border-demo demo-7"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-7"></div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      &.demo-7 {
        position: relative;
        background-color: #fff;
        height: 50px;
        width: 36px;
        border-top: 2px solid gray;
        border-bottom: 2px solid gray;
        animation: border-demo-7 3.2s infinite ease-out;
        &::before {
          position: absolute;
          top: -1px;
          left: 3px;
          content: "";
          width: 26px;
          height: 23px;
          border-width: 0 2px 1px 2px;
          border-style: solid;
          border-color: gray;
          border-radius: 0 0 100% 100% / 0 0 180% 180%;
        }
        &::after {
          position: absolute;
          bottom: -1px;
          left: 3px;
          content: "";
          width: 26px;
          height: 23px;
          border-width: 1px 2px 0 2px;
          border-style: solid;
          border-color: gray;
          border-radius: 100% 100% 0 0 / 180% 180% 0 0;
        }
        @keyframes border-demo-7 {
          0% {
            transform: rotate(0deg);
          }
          35% {
            transform: rotate(181deg);
          }
          40%, 50% {
            transform: rotate(180deg);
          }
          85% {
            transform: rotate(361deg);
          }
          90%, 100% {
            transform: rotate(360deg);
          }
        }
      }
    }
    ```
  </details>

### 跳跳跳
- 左右横跳
  <div class="jump-demo demo-1"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="jump-demo demo-1"></div>
    ```
    ```scss SCSS
    .jump-demo {
      &.demo-1 {
        position: relative;
        height: 8rem;
        width: 16rem;
        border-bottom: 2px solid gray;
        &::before {
          content: "";
          position: absolute;
          bottom: 0;
          left: 3rem;
          width: 3rem;
          height: 3rem;
          border-radius: 5px;
          background-color: gray;
          transform-origin: center center;
          animation: jump-demo-1 1.2s infinite ease-in-out alternate;
        }
        &::after {
          content: "";
          position: absolute;
          bottom: 0;
          left: 10rem;
          width: 3rem;
          height: 3rem;
          border-radius: 5px;
          background-color: gray;
          transform-origin: -80% center;
          animation: jump-demo-2 1.2s infinite linear alternate, jump-demo-2-2 1.2s infinite ease;
        }
        @keyframes jump-demo-1 {
          from {
            transform: translateX(0);
          }
          to {
            transform: translateX(7rem);
          }
        }
        @keyframes jump-demo-2 {
          0%, 20% {
            transform: rotate(0);
          }
          80%, 100% {
            transform: rotate(-180deg);
          }
        }
        @keyframes jump-demo-2-2 {
          0%, 80% {
            width: 3rem;
            height: 3rem;
          }
          82%, 86% {
            width: 3.5rem;
            height: 2.5rem;
          }
        }
      }
    }
    ```
  </details>

- 上下弹跳
  <div class="jump-demo demo-2"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="jump-demo demo-2"></div>
    ```
    ```scss SCSS
    .jump-demo {
      &.demo-2 {
        position: relative;
        width: 10rem;
        height: 0.3rem;
        margin: 10rem 0 0 0;
        background-color: gray;
        animation: jump-demo-rubberBand 0.8s infinite ease;
        &::before {
          content: "";
          position: absolute;
          left: 3.5rem;
          bottom: 0;
          width: 3rem;
          height: 3rem;
          background-color: gray;
          border-radius: 5px;
          transform-origin: center;
          animation: jump-demo-top-bottom 0.8s infinite linear, jump-demo-rotate 1.5s infinite linear;
        }
        @keyframes jump-demo-rubberBand {
          0%, 40% {
            transform: scale3d(1,1,1);
          }
          50% {
            transform: scale3d(1.25,.75,1);
          }
          60%, 100% {
            transform: scale3d(1,1,1);
          }
        }
        @keyframes jump-demo-top-bottom {
          0% {
            bottom: 5rem;
          }
          3% {
            bottom: 4.9rem;
          }
          6% {
            bottom: 4.8rem;
          }
          9% {
            bottom: 4.7rem;
          }
          12% {
            bottom: 4.6rem;
          }
          15% {
            bottom: 4.5rem;
          }
          18% {
            bottom: 4.4rem;
          }
          21% {
            bottom: 4.3rem;
          }
          24% {
            bottom: 4.2rem;
          }
          50% {
            bottom: 0.3rem;
          }
          76% {
            bottom: 4.2rem;
          }
          79% {
            bottom: 4.3rem;
          }
          82% {
            bottom: 4.4rem;
          }
          85% {
            bottom: 4.5rem;
          }
          88% {
            bottom: 4.6rem;
          }
          91% {
            bottom: 4.7rem;
          }
          94% {
            bottom: 4.8rem;
          }
          97% {
            bottom: 4.9rem;
          }
          100% {
            bottom: 5rem;
          }
        }
        @keyframes jump-demo-rotate {
          0% {
            transform: rotate(0);
          }
          100% {
            transform: rotate(270deg);
          }
        }
      }
    }
    ```
  </details>

<h2 class="to-be-continued headerlink" id="To be continued">To be continued<dot></dot></h2>
