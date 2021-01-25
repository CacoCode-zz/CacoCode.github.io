---
title: Something about CSS（持续更新⏳）
date: 2020-05-01 12:03:39
tags:
  - CSS
categories:
  - 前端
---
<link href="/scss/something-about-css.css" rel="stylesheet"></link>

> 一切来源于生活，看到或想到感兴趣的东西，仅仅用自己想要的形式表现出来。

## CSS filter
### 类`box-shadow`阴影效果

- 利用css滤镜模糊效果实现图片**自身色彩**阴影效果

  <div class="filter-shadow demo-1"></div><div class="filter-shadow demo-2"></div><div class="filter-shadow demo-3"></div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="filter-shadow demo-1"></div>
    <div class="filter-shadow demo-2"></div>
    <div class="filter-shadow demo-3"></div>
    ```
    ```scss SCSS
    .filter-shadow {
      margin: 0 10px;
      display: inline-block;
      height: 100px;
      width: 100px;
      border-radius: 50%;
      background-position: center;
      background-size: cover;
      background-repeat: no-repeat;
      position: relative;
      &::after {
        content: "";
        height: 100%;
        width: 100%;
        border-radius: 50%;
        z-index: -1;
        background: inherit;
        position: absolute;
        top: 10px;
        left: 0;
        filter: blur(10px) brightness(80%) opacity(.8);
        transform: scale(.95);
      }
      &.demo-1 {
        background-image: url("https://qiniu.zcheng.site/something-about-css-css-filter-test1.png");
      }
      &.demo-2 {
        background-image: url("https://qiniu.zcheng.site/something-about-css-css-filter-test2.png");
      }
      &.demo-3 {
        background-image: url("https://qiniu.zcheng.site/something-about-css-css-filter-test3.png");
      }
    }
    ```
  </details>

### 类Windows10毛玻璃效果

- 利用`filter`结合`background-attachment:fixed`实现

  <div class="acrylic-filer demo-1">
    <div class="acrylic">Acrylic</div>
  </div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="acrylic-filer demo-1">
      <div class="acrylic">Acrylic</div>
    </div>
    ```
    ```scss SCSS
    @mixin acrylic {
      height: 300px;
      width: 100%;
      background: url("https://qiniu.zcheng.site/something-about-css-acrylic.png") no-repeat center/contain;
      background-attachment: fixed;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .acrylic-filer.demo-1 {
      @include acrylic;
      .acrylic {
        padding: 4em 6em;
        position: relative;
        z-index: 1;
        overflow: hidden;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1), 0 1px 8px rgba(0, 0, 0, 0.2);
        &::before {
          filter: blur(10px);
          content: "";
          position: absolute;
          left: -10px;
          top: -10px;
          right: -10px;
          bottom: -10px;
          width: calc(100% + 20px);
          height: calc(100% + 20px);
          z-index: -1;
          background: url("https://qiniu.zcheng.site/something-about-css-acrylic.png") no-repeat center/contain;
          background-attachment: fixed;
        }
        &::after {
          content: "";
          position: absolute;
          top: 0;
          bottom: 0;
          left: 0;
          right: 0;
          background: rgba(255,255,255,0.7);
          z-index: -1;
        }
      }
    }
    ```
  </details>

- 使用`backdrop-filter`实现(使用起来简单，但目前兼容性较差，[查看](https://www.caniuse.com/#search=backdrop-filter)兼容性)

  <div class="acrylic-backdrop-filter demo-2">
    <div class="acrylic">Acrylic</div>
  </div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="acrylic-backdrop-filter demo-2">
      <div class="acrylic">Acrylic</div>
    </div>
    ```
    ```scss SCSS
    .acrylic-backdrop-filter.demo-2 {
      @include acrylic;
      .acrylic {
        padding: 4em 6em;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1), 0 1px 8px rgba(0, 0, 0, 0.2);
        background: rgba(255,255,255,0.7);
        -webkit-backdrop-filter: blur(10px);
        backdrop-filter: blur(10px);
        background-blend-mode: exclusion;
      }
    }
    ```
  </details>

## CSS position
### 类[pinterest](https://www.pinterest.com/)瀑布流布局

## CSS border
### 小书签
- 使用`border`结合`::before`& `::after`实现书签效果
  
  <div class="border-demo demo-1">书签</div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-1">书签</div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      /*垂直小书签*/
      &.demo-1 {
        height: 4.2rem;
        width: 2.8rem;
        font-size: 1.5rem;
        line-height: 2rem;
        &::after {
          content: "";
          position: absolute;
          left: 0;
          top: 100%;
          border: 1.4rem solid gray;
          border-top-width: 0;
          border-bottom-color: transparent;
          border-bottom-left-radius: 3px;
          border-bottom-right-radius: 3px;
        }
      }
    }
    ```
  </details><br>

  <div class="border-demo demo-2">书签</div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-2">书签</div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      /*水平小书签*/
      &.demo-2 {
        height: 2.8rem;
        width: 4.2rem;
        font-size: 1.5rem;
        line-height: 2.8rem;
        &::after {
          content: "";
          position: absolute;
          left: 100%;
          top: 0;
          border: 1.4rem solid gray;
          border-left-width: 0;
          border-right-color: transparent;
          border-top-right-radius: 3px;
          border-bottom-right-radius: 3px;
        }
      }
    }
    ```
  </details><br>

  <div class="border-demo demo-3">我是一个小书签</div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-3">我是一个小书签</div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      /*水平带小折角书签*/
      &.demo-3 {
        height: 2.8rem;
        width: 12rem;
        font-size: 1.5rem;
        line-height: 2.8rem;
        &::after {
          content: "";
          position: absolute;
          left: 100%;
          top: 0;
          border: 1.4rem solid gray;
          border-left-width: 0;
          border-right-color: transparent;
          border-top-right-radius: 3px;
          border-bottom-right-radius: 3px;
        }
        &:before {
          content: "";
          position: absolute;
          left: 0;
          top: -1.6rem;
          border: 0.8rem solid #555555;
          border-top-color: transparent;
          border-left-color: transparent;
        }
      }
    }
    ```
  </details><br>

### 奇形怪状

- 使用`border`结合`::before`& `::after`实现平行四边形效果

  <div class="border-demo demo-4"></div><br>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="border-demo demo-4"></div>
    ```
    ```scss SCSS
    .border-demo {
      position: relative;
      background: gray;
      color: #fff;
      text-align: center;
      cursor: pointer;
      &.demo-4 {
        @mixin border-demo-4 {
          content: "";
          position: absolute;
          border: 1rem solid gray;
          border-bottom-left-radius: 3px;
        }
        width: 0rem;
        height: 0rem;
        position: relative;
        margin-left: 20px;
        &::before {
          @include border-demo-4;
          right: 0;
          border-top-color: transparent;
          border-left-color: transparent;
          border-bottom-left-radius: 3px;
        }
        &::after {
          @include border-demo-4;
          left: 0;
          border-bottom-color: transparent;
          border-right-color: transparent;
          border-top-right-radius: 3px;
        }
      }
    }
    ```
  </details>

### 阳光阴影效果 (test)

  <div class="shadow-demo demo-1">
    <div class="card-1">
      <div class="card-title">Oranges</div>
      <div class="card-footer">zcheng.site</div>
    </div>
    <div class="shadow-demo-item ellipse-900"></div>
  </div>

  <details>
    <summary>显示代码</summary>
    ```html HTML
    <div class="shadow-demo demo-1">
      <div class="card-1">
        <div class="card-title">Oranges</div>
        <div class="card-footer">zcheng.site</div>
      </div>
      <div class="shadow-demo-item ellipse-900"></div>
    </div>
    ```

    ```scss SCSS
    .shadow-demo {
      &.demo-1 {
        position: relative;
        width: 100%;
        height: 50rem;
        background-image: url("https://qiniu.zcheng.site/something-about-css-panel-bg.png");
        border-radius: 10px;
        z-index: 9;
        overflow: hidden;
        .card-1 {
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          width: 40%;
          height: 40rem;
          background-color: #fff;
          box-shadow: 2px 2px 5px 0px #1f1f1f;
          .card-title {
            position: absolute;
            top: 30%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 36px;
            font-weight: bold;
          }
          .card-footer {
            position: absolute;
            bottom: 10%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 14px;
            font-weight: bold;
          }
        }
        .shadow-demo-item {
          &.ellipse-900 {
            position: absolute;
            top: 0;
            left: 0;
            bottom: 0;
            right: 0;
            z-index: 10;
            transform: rotate(-66deg);
            transform-origin: center;
            filter: blur(15px);
            background: linear-gradient(transparent 60%, rgba(0, 0, 0, 0.6) 15%);
            background-size: 100% 18%;
          }
        }
      }
    }

    ```
  </details>

<h2 class="to-be-continued headerlink" id="To be continued">To be continued<dot></dot></h2>
