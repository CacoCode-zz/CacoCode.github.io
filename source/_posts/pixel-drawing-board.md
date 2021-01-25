---
title: Canvas学习与实践 - 基于Canvas的像素画板实现
date: 2020-09-27 10:42:38
tags:
  - HTML
  - Canvas
categories:
  - 前端
---

最近在学习Canvas，想到之前说到的[Css像素画](./pixel-icon-design.html)能不能基于Canvas来设计，正好Canvas处理与绘画图像很强大，并且可以精确到控制每一个像素，还挺适合作为一个像素画板的。

### 效果演示

演示画一个bilibili小电视

![bilibili小电视](https://qiniu.zcheng.site/pixel-drawing-board/draw_demo.gif)

### 设计思路

既然是画像素画的，自然不能跟普通画板一样线条边缘平滑，而是反其道而行之。通过放大每一个像素，营造一种像素风的感觉。建立一定尺寸的Canvas画布，放大每一个像素，通过鼠标操作每一个像素块颜色来完成一张像素画的制作。

### 具体实践

#### 生成初始像素画板

想象像素画板每一个网格都是一个像素点，对每个像素点进行放大，方便鼠标操作。

```html HTML
<div class="drawing-board-wrapper">
  <canvas id="drawing-board"></canvas>
</div>
```

下面对Canvas设置100%的宽高以放大整个画板，并需要设置`image-rendering: pixelated;`来放大每一个像素点，否者单纯放大Canvas会造成放大后模糊的现象。

```css CSS
.drawing-board-wrapper {
  max-width: 937px;
  max-height: 937px;
}
#drawing-board {
  width: 100%;
  height: 100%;
  image-rendering: pixelated;
}
```

整个画布宽高我设置为浏览器窗口高度减去100px，默认画布网格行数与列数为32，每个网格宽高设置为1px，并将每个网格X，Y偏移值，宽高，颜色存入`initGridList`中，方便后续操作。`generateInitialGrids`方法判断当前网格排序为奇数或偶数来赋予相应的背景颜色，将数据存入`initGridList`列表。`initDrawingBoard`方法遍历`initGridList`列表生成每一个小网格来组成整个画板。

```js JS
// 画布大小
let gridLayoutWH = document.body.clientHeight - 100

// 行列网格数
let gridNum = 32

// 单个网格宽高
let wh = 1

// 初始画板中的所有网格列表
let initGridList = []

// 生成初始黑白网格列表
function generateInitialGrids () {
  // 当前网格序号 - 用来计数 - 判断奇和偶。奇：#d9d9d9；偶：#fff
  let num = 0
  initGridList = []

  for (let i = 0; i < gridNum; i++) {
    for (let j = 0; j < gridNum; j++) {
      num ++
      if (gridNum % 2 === 0) {
        if (i % 2 === 0) {
          if (num % 2 === 0) {
            defaultGridColor = '#fff'
          } else {
            defaultGridColor = '#d9d9d9'
          }
        } else {
          if (num % 2 === 0) {
            defaultGridColor = '#d9d9d9'
          } else {
            defaultGridColor = '#fff'
          }
        }
      } else {
        if (num % 2 === 0) {
          defaultGridColor = '#fff'
        } else {
          defaultGridColor = '#d9d9d9'
        }
      }

      initGridList.push(
        {
          x: gridLayoutWH / gridNum * j,
          y: gridLayoutWH / gridNum * i,
          wh: gridLayoutWH / gridNum,
          color: defaultGridColor
        }
      )
    }
  }
}

// 初始化画板
function initDrawingBoard() {
  // 设置canvas宽高
  canvas.setAttribute('width', gridNum)
  canvas.setAttribute('height', gridNum)
  for (let grid of initGridList) {
    ctx.beginPath()
    ctx.fillStyle = grid.color
    ctx.fillRect(grid.x / (gridLayoutWH / gridNum) , grid.y / (gridLayoutWH / gridNum), wh, wh)
  }
}

generateInitialGrids()
initDrawingBoard()
```

#### 监听鼠标事件

初始画板生成后，需要通过鼠标在画板上面进行绘画，所以我们需要设置相应的点击、移动、聚焦等鼠标动作来绘画，清除、选中网格。

- `handleEvent`用来监听鼠标执行了哪种操作，并调用相应的方法。
- `hoverGrids`方法用来通知当前鼠标悬停在哪个网格上，并通过加深网格背景颜色来告知用户。
- `clearHoverGrids`方法清除上一个网格悬停效果。
- `drawingGrids`方法通过点击鼠标左键触发，并改变当前鼠标所在网格背景颜色。
- `eraseGrids`方法通过点击鼠标右键触发，并清除当前鼠标所在网格背景颜色。
- `draw`方法能精确定位鼠标所在网格的序号，并改变当前网格颜色。


```js JS
// 绘画状态
let drawing = false

// handleEvent方法监听鼠标事件并进行处理。
function handleEvent(e) {
  let x = e.offsetX,
    y = e.offsetY

  if (drawing) {
    // 鼠标左键
    if (e.buttons === 1) {
      drawingGrids(x, y)
    // 鼠标右键
    } else {
      eraseGrids(x, y)
    }
  } else {
    hoverGrids(x, y)
  }
}

// 鼠标悬停效果
function hoverGrids(x, y) {
  clearHoverGrids()
  draw(x, y, hoverColor, 'hover')
}

// 清除上一帧鼠标悬停效果
function clearHoverGrids() {
  if (!drawing) {
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    initDrawingBoard()
  }
}

// 在画板中按住鼠标左键拖动或点击画板网格
function drawingGrids(x, y) {
  draw(x, y, drawingColor, 'leftClick')
}

// 擦除网格颜色 - 橡皮擦功能
function eraseGrids(x, y) {
  draw(x, y, eraseColor, 'rightClick')
}

// 网格上色
function draw(x, y, c, e) {
  for (let i = 0; i < initGridList.length; i++) {
    if ((x >= initGridList[i].x) &&
      (y >= initGridList[i].y) &&
      (x < initGridList[i].x + initGridList[i].wh) &&
      (y < initGridList[i].y + initGridList[i].wh)) {
      let axisX = initGridList[i].x / (gridLayoutWH / gridNum)
      let axisY = initGridList[i].y / (gridLayoutWH / gridNum)

      // 获取当前鼠标所在网格X，Y轴
      currGridPosition.innerHTML = `<span>X: ${Math.ceil(axisX + 1)},
        Y: ${Math.ceil(axisY + 1)}</span>`

      if (e === 'leftClick') {
        initGridList[i].color = c
      } else if (e === 'rightClick') {
        // 当前点击网格行数 - 奇或偶
        if (Math.ceil(axisY + 1) % 2 === 0) {
          if(Math.ceil(axisX + 1) % 2 === 0) {
            c = '#d9d9d9'
          }
        } else {
          if(Math.ceil(axisX + 1) % 2 !== 0) {
            c = '#d9d9d9'
          }
        }
        initGridList[i].color = c
      }

      ctx.beginPath()
      ctx.fillStyle = c
      ctx.fillRect(axisX , axisY, wh, wh)
    }
  }
}

canvas.addEventListener('mousemove', function(e) {
  handleEvent(e)
})

canvas.addEventListener('mouseout', function(e) {
  clearHoverGrids(e)
})

canvas.addEventListener('mousedown', function(e) {
  drawing = true
  handleEvent(e)
})

canvas.addEventListener('mouseup', function(e) {
  drawing = false
})
```

#### 画板设置

画板设置能提供部分接口来修改画板绘画属性，例如画板网格数、画笔颜色、画笔粗细等配置。

功能列表

- [x] 画板网格数
- [x] 画笔颜色
- [ ] 画笔粗细
- [ ] 快捷工具
- [x] 预览功能 [2020.10.16]
- [x] 导出图片 [2020.10.16]
- [ ] 导出CSS box-shadow

### 未解决的问题 & bugs

- 绘画时，鼠标移动过快，出现线条不连贯中断问题。
- 等等

### demo & 项目地址

- [demo在线](https://pixel.zcheng.site/)
- [github源码](https://github.com/zchengsite/pixel-drawing-board)

### 未完待续
