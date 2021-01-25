---
title: SCSS语法学习与实践 - 将现有项目CSS整合成SCSS
date: 2020-05-16 09:44:48
tags:
  - SCSS
categories:
  - 前端
---

[Scss](https://sass.bootcss.com/documentation/syntax)是[Sass](https://sass-lang.com/)的一种新语法。相较于传统的Css3，其引入了变量、函数、继承等一些新的概念，帮助我们更方便的书写Css。今天正好系统性学习下Scss，把博客Css代码整合成Scss，也算实践练手了。😋

---

拿以下HTML代码举例
```html
<div class="transition-demo demo-1-parent">
  <div class="demo-1-child">
    <a href="#">zcheng.site</a>
  </div>
</div>

<div class="filter-shadow demo-1"></div>
<div class="filter-shadow demo-2"></div>
<div class="filter-shadow demo-3"></div>
```

## 嵌套规则
```scss
.transition-demo {
  // some css style
  .demo-1-child {
    // some css style
    a {
      // some css style
    }
  }
}
```

## 引用嵌套中的父级选择器
在scss中进行嵌套时，可以使用`&`表示父级选择器。以下scss代码中`&.demo-1-parent`表示的是找到它的父级选择器`.transition-demo`，编译成css可以理解为`.transition-demo.demo-1-parent`。同理，下面的`&:hover`表示找到父级`a`，编译成css为`a:hover`。
```scss
.transition-demo {
  // some css style
  &.demo-1-parent {
    // some css style
    .demo-1-child {
      // some css style
      a {
        // some css style
        &:hover {
          // some css style
        }
      }
    }
  }
}
```

## 嵌套属性规则
在scss中可以将相同的css属性部分提取出来，如`font`、`border`属性。
```scss
.transition-demo {
  font {
    family: fantasy;
    size: 1rem;
    style: normal;
    weight: bold;
  }
  border: 1px solid gray {
    radius: 10px;
    left: 0;
    right: 0;
  }
}
```
scss是以`{}`和`;`来识别属性和子属性，以`;`结尾识别为一个子属性，以`{`结尾即识别为一个嵌套着子属性的大属性。通过`-`来拼接属性与它所嵌套的子属性来达到效果。

## 声明与使用变量
scss使用`$`来声明一个变量
```scss
$fontSize: 16px; // 声明一个变量
.transition-demo {
  font-size: $fontSize; // 使用变量
}
```
### 优先级问题
```scss
$fontSize: 16px; // 声明一个变量
$fontSize: 14px; // 重复声明变量
.transition-demo {
  font-size: $fontSize; // 使用变量
}
```
编译为css后
```scss
.transition-demo {
  font-size: 14px; // 结果为14px
}
```
声明相同变量，后面声明的会覆盖前面声明的。

### 作用域问题
scss可以在嵌套选择器内部声明变量，那么这个变量只能用于这个嵌套选择器内部。在声明选择器外部使用无效。
```scss
.transition-demo {
  $fontSize: 16px; // 在一个选择器内部声明一个变量
  font-size: $fontSize; // 使用变量，有效
  .demo-1-child {
    font-size: $fontSize; // 使用变量，有效
  }
}

.filter-shadow {
  font-size: $fontSize; // 使用变量，无效
}
```

## 声明与使用函数
scss使用`@mixin`来声明一个函数，使用`@include`来使用声明的函数。
如果css代码中有重复使用的代码片段，可以使用函数将它们封装起来复用。
```scss
// 声明一个名为filterShadow的函数，定义一个叫$transform的参数，可根据情况来传递参数。
@mixin filterShadow($transform) {
  content: "";
  position: absolute;
  background: #262626;
  transform: $transform;
  transition: transform 0.3s;
}

.filter-shadow {
  &.demo-1 {
    &::before, &::after {
      @include filterShadow(scaleX(0)) // 使用函数，传递参数
    }
  }
  &.demo-2 {
    &::before, &::after {
      @include filterShadow(scaleY(0)) // 使用函数，传递参数
    }
  }
}
```
### 作用域问题
scss允许在嵌套的选择器中声明函数，此时函数作用域与声明变量同理。

## 定义与使用继承
scss使用`%`定义一个需要被继承的样式，通过`@extend`来使用继承。
```scss
// 定义一个名为filterStyle的继承样式
%filterStyle {
  content: "";
  position: absolute;
  background: #262626;
  transform: $transform;
  transition: transform 0.3s;
}

.filter-shadow {
  &.demo-1 {
    @extend %filterStyle; // 继承样式
    left: 0;
    right: 0;
  }
  &.demo-2 {
    @extend &.demo-1; // 继承样式
  }
}
```

The End!
