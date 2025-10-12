---

hide:
  - path
  - toc
  - feedback
  - navigation
  - footer
---

# 
<!-- Lorem ipsum dolor sit amet -->

___Life is much harder than you think, but you have to stick to it, or you'll never be able to ground off___

<!-- ![Alt text](Top/E.png) -->

<style>
  /* 嵌入 style.css 内容 */
  html, body {
    margin: 0;
    padding: 0;
  }
  canvas {
    display: block;
  }
</style>

<script>
  // 嵌入 sketch.js 的内容
  let t = 0; // time
  let complexitySlider;
  let speedSlider;
  let shapeSlider;
  let sizeSlider;
  let focusSlider;

  const sliderHeight = 25;
  const sliderWidthMod = 0.15;

  const complexityStart = 5;
  const complexityMin = 5;
  const complexityMax = 20;

  const speedStart = 60;
  const speedMin = 20;
  const speedMax = 70;

  const shapeStart = 99;
  const shapeMin = 1;
  const shapeMax = 200;

  const sizeStart = 3;
  const sizeMin = 1;
  const sizeMax = 4;

  const focusStart = 1;
  const focusMin = 0.5;
  const focusMax = 4;

  function setup() {
    createCanvas(windowWidth * 0.8, windowHeight * 0.5);
    // background(100);

    // 定义滑块组的宽度、右侧偏移量和顶部偏移量
    const sliderGroupWidth = windowWidth * sliderWidthMod; // 滑块组的宽度
    const sliderGroupX = windowWidth - sliderGroupWidth - 40; // 距离右侧20px的间距
    const sliderGroupY = 160; // 距离顶部20px的间距

    // 初始化滑块并设置它们的尺寸和位置
    complexitySlider = createSlider(complexityMin, complexityMax, complexityStart);
    complexitySlider.position(sliderGroupX, sliderGroupY);
    complexitySlider.size(sliderGroupWidth, sliderHeight);

    speedSlider = createSlider(speedMin, speedMax, speedStart);
    speedSlider.position(sliderGroupX, sliderGroupY + sliderHeight * 1);
    speedSlider.size(sliderGroupWidth, sliderHeight);

    shapeSlider = createSlider(shapeMin, shapeMax, shapeStart);
    shapeSlider.position(sliderGroupX, sliderGroupY + sliderHeight * 2);
    shapeSlider.size(sliderGroupWidth, sliderHeight);

    sizeSlider = createSlider(sizeMin, sizeMax, sizeStart, 0.2);
    sizeSlider.position(sliderGroupX, sliderGroupY + sliderHeight * 3);
    sizeSlider.size(sliderGroupWidth, sliderHeight);

    focusSlider = createSlider(focusMin, focusMax, focusStart, 0.1);
    focusSlider.position(sliderGroupX, sliderGroupY + sliderHeight * 4);
    focusSlider.size(sliderGroupWidth, sliderHeight);
  }

  function a(x, y, d = mag(k = x / 8 - 25, e = y / 8 - 25) ** 2 / 99) {
    // 获取偏移后以中心对齐的坐标
    const q = (x / sizeSlider.value() + k * 0.5 / cos(y * 5) * sin(d * d - t));
    const c = d / 2 - t / 8;

    return [
        q * sin(c) + e * sin(d + k - t),    // 移除 windowWidth*0.5 偏移量
        (q + y / 8 + d * 9) * cos(c)        // 移除 windowHeight*0.5 偏移量
    ];
}

draw = function() {
    if (t === 0) {
        createCanvas(windowWidth, windowHeight);
    }
    clear();  // 设置背景透明
    // stroke(255, 96);  // 设置笔触颜色
    stroke(128, 0, 128);  // 设置画笔颜色为紫金色
    strokeWeight(focusSlider.value());
  
    t += PI / speedSlider.value();

    // 将坐标系的原点移动到画布的中心
    translate(windowWidth / 3, windowHeight / 3.5);

    for (let y = shapeSlider.value(); y < shapeSlider.value() + 200; y += complexitySlider.value()) {
        for (let x = shapeSlider.value(); ++x < shapeSlider.value() + 200;) {
            point(...a(x, y));
        }
    }
};
</script>