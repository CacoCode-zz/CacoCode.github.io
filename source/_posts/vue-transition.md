---
title: Vue transition 折叠类动画自动获取隐藏层高度以及手风琴效果实现
date: 2019-07-13 12:37:58
tags:
  - CSS
  - VUE
categories:
  - 前端
---

前段时间，在解决左侧导航栏激活问题时，发现折叠动画过渡不是很平滑，并且还伴随着一些bug。使用起来不是很流畅，于是在解决完激活bug后，我决定把动画效果也解决掉。

---

### 要解决的问题
**如下图**

![最初的样式!](https://qiniu.zcheng.site/vue-transition-before1.gif)
不知细心的你有没有发现什么问题？
当点击折叠展开`menu-one`栏目的时候没有什么问题，但点击折叠展开`mene-two`栏目，当动画定格在`menu-two-2`选项的时候，`menu-two-3`及以下的部分直接立即完成了过渡。为什么会这样？下面说原因。

**上图实现代码：**（以css代码为主）

```html HTML
  <transition name="sub-menu">
    <ul
      class="site-menu-sub"
      v-show="item.children && isOpen === aindex">
      <li
        v-for="(child, index) in item.children"
        :key="index"
        class="site-menu-item">
        <router-link
          class="animsition-link"
          :to="`/${child.href}`">
        </router-link>
      </li>
    </ul>
  </transition>
```

```css CSS
  .site-menu-sub {
    padding: 0px;
  }
  .sub-menu-enter-active,
  .sub-menu-leave-active {
    transition: all .3s ease;
    height: 70px; /* 这里70px为menu-one子菜单的高度 */
  }
  .sub-menu-enter,
  .sub-menu-leave-to
  {
    opacity: 0;
    height: 0px;
  }
```
这里因为我把过渡完成时（完全展开时）的`height`设置为70px，`height` 从0 ~ 70px 之间完成了过渡，但高于70px的元素就会有问题，高度超出70px部分会立即完成过渡，没有平滑过渡效果。这里我设置的70px高度正好为`muen-one`展开的高度，这就是为什么`menu-one`展开折叠没问题，而`menu-two`会有问题。同样的，如果我把`height`设置为`menu-two`展开的高度（210px），那么`menu-one`展开折叠动画就会有问题。这里我不举例说明，可以自己尝试。

**上面的例子说明：**
通过`transition`过渡去控制变化元素高度或者宽度时：
  - 多个折叠类元素高度或宽度不一样时，不能直接将高或宽写为固定值（写死）。
  - 折叠类元素不确定高或宽的情况下，也不能直接将高或宽写为固定值（写死）。

---

### 折衷的解决方案

**思路**

>通过`max-height` 或 `max-width` 去控制元素的最大高宽。

![折衷的解决方案的样式!](https://qiniu.zcheng.site/vue-transition-before2.gif)

看出问题来了吗？过渡效果好像变得很快，这是因为`max-height`设置成了1000px，不过比最开始那个效果要好一点。

**折衷解决方案代码**
```css CSS
  .site-menu-sub {
    padding: 0px;
  }
  .sub-menu-enter-active,
  .sub-menu-leave-active {
    transition: all .3s ease;
    max-height: 1000px; /* 这里将最大高度设置为折叠元素不可能超越的高度 比如1000px */
  }
  .sub-menu-enter,
  .sub-menu-leave-to
  {
    opacity: 0;
    max-height: 0px;
  }
```

html代码部分没有改变，仅仅将`height`改为`max-height`了，并且将`max-height`设置为折叠元素不可能超越的高度。如果`max-height`设置的太小，比展开后元素高度低。那展开后样式会出现问题，展示不完全。并且这种解决方案没达到最完美效果，代码维护成本也大。

---

### 完美的解决方案

如下图

![完美的解决方案的样式!](https://qiniu.zcheng.site/vue-transition-before3.gif)

**思路**

>通过JavaScript 钩子的方式获取隐藏层的高度，动态去改变隐藏层的高度

**具体实现方案和代码如下：**

新建`collapse-transition.js`（名字自己定义）作为函数式组件被引用，代码如下：

```javascript collapse-transition.js
  // collapse-transition.js
  const transitionStyle = '0.3s height ease-in-out';
  const Transition = {
    beforeEnter(el) {
      el.style.transition = transitionStyle;
      if (!el.dataset) el.dataset = {};

      el.style.height = 0;
    },

    enter(el) {
      if (el.scrollHeight !== 0) {
        el.style.height = `${el.scrollHeight}px`;
      } else {
        el.style.height = '';
      }
      el.style.overflow = 'hidden';
    },

    afterEnter(el) {
      el.style.transition = '';
      el.style.height = '';
    },

    beforeLeave(el) {
      if (!el.dataset) el.dataset = {};
      el.style.height = `${el.scrollHeight}px`;
      el.style.overflow = 'hidden';
    },

    leave(el) {
      if (el.scrollHeight !== 0) {
        el.style.transition = transitionStyle;
        el.style.height = 0;
      }
    },

    afterLeave(el) {
      el.style.transition = '';
      el.style.height = '';
    },
  };

  export default {
    name: 'CollapseTransition',
    functional: true,
    render(h, { children }) {
      const data = {
        on: Transition,
      };
      return h('transition', data, children);
    },
  };
```
在需要的页面引入：
```html mune.vue
<script>
  import CollapseTransition from '@/utils/collapse-transition'; // 本人将collapse-transition.js 放置在工具类utils文件夹

  export default {
    components: {
      'collapse-transition': CollapseTransition,
    },
  };
</script>
```
使用：
```html mune.vue
  <collapse-transition>
    <ul
      class="site-menu-sub"
      v-show="item.children && isOpen === aindex">
      <li
        v-for="(child, index) in item.children"
        :key="index"
        class="site-menu-item">
        <router-link
          class="animsition-link"
          :to="`/${child.href}`">
        </router-link>
      </li>
    </ul>
  </collapse-transition>
```
将之前的css删除，不再需要了。

到这一步自动获取元素高度的问题解决了，可以说是“一劳永逸”了。不管隐藏层高度多高，动画都没有问题。

---

### 手风琴模式

不知你们发现了一个问题没，以上实现效果图有个过渡动画bug，就是手风琴模式（只保持一个子菜单的展开）过渡效果没出来。在`menu-two`展开状态下点击`折叠的menu-one`，我们发现`menu-two`直接关闭，而没有过渡动画。

解决方案：

```javascript collapse-transition.js
  beforeLeave(el) {
    if (!el.dataset) el.dataset = {};
    el.style.display = 'block'; // 添加这一行
    el.style.height = `${el.scrollHeight}px`;
    el.style.overflow = 'hidden';
  },
```

在`beforeLeave`钩子中加入 `display: 'block'` 如上图。

实现效果图
![手风琴模式!](https://qiniu.zcheng.site/vue-transition-after.gif)

到这里就彻底解决了vue transition折叠展开过渡动画的问题。

---

### 具体参考资料
- [vuejs如何实现这样的展开收起动画？](https://segmentfault.com/q/1010000011359250)
- [vue 官方中文文档-进入/离开 & 列表过渡-JavaScript 钩子](https://cn.vuejs.org/v2/guide/transitions.html#JavaScript-%E9%92%A9%E5%AD%90)
- [element-ui collapse-transition动画源码](https://github.com/ElemeFE/element/blob/dev/src/transitions/collapse-transition.js)

### 其他说明
- gif录制过程中难免出现卡顿掉帧现象，本页面所有效果图不能代表实际实现效果。
- gif录制软件[ScreenToGif](https://github.com/NickeManarin/ScreenToGif)。

### demo & 项目地址
- [demo](https://zchengsite.github.io/nuxt-admin/)
- [项目地址](https://github.com/zchengsite/nuxt-admin)
- [menu组件代码](https://github.com/zchengsite/nuxt-admin/blob/master/components/layouts/Menubar/NewMenu.vue)
- [transition动画代码](https://github.com/zchengsite/nuxt-admin/blob/master/utils/collapse-transition.js)

The End😀
