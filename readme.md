因为canvas是基于像素的绘图，我们在pc端封装的canvas绘制方法如果想直接迁移至移动端并实现移动端适配，可以使用如下思路：

使用`transform`的`scale`对画布进行缩放，缩放比例的计算并不难，思路就是canvas（像素大小）占手机dip的比例等于设计稿上canvas占设计稿的比例，并且移动设备上canvas的像素大小可以用pc端的canvas大小✖️缩放比例来表示，即：

~~~
pc端canvas图像宽度(像素大小) * 缩放比例 / 移动设备的dip宽度 === 设计稿上canvas部分的宽度 / 设计稿宽度
~~~

综上可得适配方法`adaptMobile`：

~~~js
/**
 * @param {number} designWidth: 设计稿上的canvas宽度
 * @param {number} width: 设计稿的宽度
 * @param {Element} canvasElm: canvas元素
 */
function adaptMobile(designWidth, width, canvasElm) {
  const ratio = designWidth / width; // 目标canvas占屏幕的宽度比例
  const screenPx = screen.width; // 当前设备的宽度dip像素数
  const canvasWidth = parseInt(canvasElm.style.width); // canvas的宽度
  canvasElm.style.transformOrigin = '0 0';
  canvasElm.style.transform = `scale(${ratio * screenPx / canvasWidth})`;
}
~~~

`canvasElm`即用pc端的绘制方法所绘制的画布，但当前还存在一个问题，因为pc端绘制时canvas的像素大小（宽度）可能超出移动设备的dip宽度，虽然经过`transform`缩放后画布已经完全在手机屏幕之内，但是熟悉浏览器渲染原理的朋友肯定知道，`transform`是在渲染的最后一步完成的，不参与render tree的构建以及布局绘制等等前面的逻辑，所以这就导致移动端出现滚动条。

解决方案很简单：给canvas的dom元素外面套一个空`<div>`，给这个div设置一个`overflow: hidden`即可解决。

综上我们就完成了pc端绘制的canvas图像无痛迁移到移动端实现适配的流程。

给个demo，如下`adaptMobile(375, 750, canvasElm);`即等价于让canvas元素`width: 50vw`，可以切换不同型号的移动设备感受：

~~~html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .canvas-container {
        overflow: hidden;
      }
    </style>
  </head>
  <body>
    <div class="canvas-container">
      <canvas id="canvasElm"></canvas>
    </div>
    <script>
      const canvas = document.getElementById('canvasElm');
      canvasElm.width = 300;
      canvasElm.height = 300;
      canvasElm.style.width = 500 + 'px';
      canvasElm.style.border = `1px solid red`;
      const ctx = canvasElm.getContext('2d');
      ctx.fillStyle = 'pink';
      ctx.fillRect(0, 0, 150, 150);
      /**
       * @param {number} designWidth: 设计稿上的canvas宽度
       * @param {number} width: 设计稿的宽度
       * @param {Element} canvasElm: canvas元素
       */
      function adaptMobile(designWidth, width, canvasElm) {
        const ratio = designWidth / width; // 目标canvas占屏幕的宽度比例
        const screenPx = screen.width; // 当前设备的宽度dip像素数
        const canvasWidth = parseInt(canvasElm.style.width); // canvas的宽度
        canvasElm.style.transformOrigin = '0 0';
        canvasElm.style.transform = `scale(${ratio * screenPx / canvasWidth})`;
      }
      adaptMobile(375, 750, canvasElm);
    </script>
  </body>
</html>
~~~

plus：这里有人反应过来会问，为啥不直接设置`canvas`元素宽度为`50vw`，因为`canvas`只支持通过`canvasElm.style.width`来设置像素大小hh。

并且这里还有个可能让你迷惑的点就是，为啥不直接操作`canvasElm.style.width`完成移动端适配，也就是说计算移动设备上应该要画多少像素，完全没问题，但是业务场景所需，可以理解为我们业务中pc端的绘制方法内包含了对`canvasElm.style.width`的设置，并且这个方法相当于一个黑盒，我们无法操作。