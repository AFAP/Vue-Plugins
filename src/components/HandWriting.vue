<template>
  <div class="handwriting">
    <div class="handwriting-header">

      <img :class="['handwriting-img-back', pointLines.length == 0 ? 'handwriting-img-disable' : '']" alt="返回上一步" src="~@/assets/image/handwriting-back.png" @click="backPreStep()">
      <img class="handwriting-img-setting" alt="设置颜色和笔刷" src="~@/assets/image/handwriting-setting.png" @click="showSetting = !showSetting">
      <div :class="[pointLines.length == 0 ? 'handwriting-img-disable' : '']">完成</div>

    </div>

    <canvas id="canvas-handwriting" class="handwriting-canvas"></canvas>

    <div class="handwriting-setting" v-if="showSetting">
      <div>
        笔触:
        <div class="handwriting-setting-brush-container">
          <div :class="['handwriting-setting-brush', lineWidth == 4 ? 'handwriting-setting-brush-selected' : '']" @click="changeLineWidth(4)">细</div>
          <div :class="['handwriting-setting-brush', lineWidth == 8 ? 'handwriting-setting-brush-selected' : '']" @click="changeLineWidth(8)">中</div>
          <div :class="['handwriting-setting-brush', lineWidth == 12 ? 'handwriting-setting-brush-selected' : '']" @click="changeLineWidth(12)">粗</div>
        </div>
      </div>

      <div>
        颜色：
        <div class="handwriting-palette-container">
          <div v-for="item in COLORS" :key="item" :class="['handwriting-palette-dot', lineColor == item ? 'handwriting-palette-dot-selected' : '']" :style="{backgroundColor: item}" @click="changeLineColor(item)"></div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
Array.prototype.clone = function () {
  return [].concat(this);
  //或者 return this.concat();
}
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.isControl = false;
    this.time = Date.now();
    this.lineWidth = 0;
    this.isAdd = false;
  }
}

class Line {
  constructor() {
    this.points = new Array();
    this.changeWidthCount = 0;
    this.lineWidth = 8;
    this.lineColor = "#222222";
  }
}

export default {
  name: 'HandWriting',
  props: {
    msg: String,

  },
  data() {
    return {
      COLORS: ["#222222", "#F3100F", "#F7A311", "#21CD38", "#359FF4", "#5D5FE9"],
      lineWidth: 8,
      lineColor: "#222222",
      canvas: null,
      ctx: null,
      showSetting: false,
      isDown: false, // 鼠标或手势点按了
      pointLines: [], // 线条数组
      line: new Line(), // 当前线条
      preTime: 0,
      k: 0.5,
      begin: null,
      middle: null,
      end: null
    };
  },
  mounted() {
    let canvas = document.getElementById('canvas-handwriting');
    let ctx = canvas.getContext("2d");
    ctx.lineWidth = this.lineWidth; //设置画笔粗细
    ctx.strokeStyle = this.lineColor;
    ctx.fillStyle = this.lineColor;

    ctx.lineJoin = "round"; //设置画笔轨迹基于圆点拼接


    let width = canvas.clientWidth;
    let height = canvas.clientHeight;


    console.log(height, width)

    if (window.devicePixelRatio) {
      canvas.style.width = width + "px";
      canvas.style.height = height + "px";
      canvas.height = height * window.devicePixelRatio;
      canvas.width = width * window.devicePixelRatio;
      ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
    }



    canvas.onmousedown = this.beginDraw;
    canvas.onmousemove = this.moving;
    canvas.onmouseup = this.endDraw;

    canvas.addEventListener("touchstart", this.beginDraw);
    canvas.addEventListener("touchmove", this.moving);
    canvas.addEventListener("touchend", this.endDraw);

    this.canvas = canvas;
    this.ctx = ctx;

  },
  methods: {
    changeLineWidth(val) {
      this.lineWidth = val;
    },
    changeLineColor(val) {
      this.lineColor = val;
    },
    backPreStep() {
      this.pointLines.splice(this.pointLines.length - 1, 1);
      this.line = new Line();
      this.draw(false)
    },
    isTouchEvent(event) {
      return event.type.indexOf('touch') >= 0;
    },
    beginDraw(event) {
      console.log('beginDraw')
      if (this.showSetting) {
        this.showSetting = false;
        return;
      }

      this.isDown = true;
      this.line = new Line();
      this.line.lineWidth = this.lineWidth;
      this.line.lineColor = this.lineColor;
      this.addPoint(this.GetPos(event));
      this.preTime = Date.now();
    },
    endDraw(event) {
      if (!this.isDown) {
        return;
      }
      console.log('endDraw')
      this.isDown = false;
      event.preventDefault();

      this.addPoint(this.GetPos(event));
      this.draw(true);
      this.pointLines.push(this.line);

      this.begin = null;
      this.middle = null;
      this.end = null;
    },
    moving(event) {
      // 必须有起手
      if (!this.isDown) {
        return;
      }
      event.preventDefault();
      let xy = this.GetPos(event);
      this.addPoint(xy);
      this.draw(false)
    },
    GetPos(event) {
      // console.log(event)
      let x = 0, y = 0;
      if (this.isTouchEvent(event)) {
        // console.log(event)
        if (event.targetTouches.length == 0) {
          if (this.line.points.length > 0) {
            return this.line.points[this.line.points.length - 1];
          } else {
            return new Point(0, 0)
          }
        } else {
          let touch = event.targetTouches[0];
          x = touch.clientX - touch.target.offsetLeft;
          y = touch.clientY - touch.target.offsetTop;
        }
      } else {
        x = event.offsetX;
        y = event.offsetY;
      }
      return new Point(x, y)
    },
    addPoint(p) {
      if (this.line.points.length >= 1) {
        let last_point = this.line.points[this.line.points.length - 1]
        let distance = this.z_distance(p, last_point);
        if (distance < 10) {
          return;
        }
      }

      if (this.line.points.length == 0) {
        this.begin = p;
        p.isControl = true;
        this.pushPoint(p);
      } else {
        this.middle = p;
        let controlPs = this.computeControlPoints(this.k, this.begin, this.middle, null);
        this.pushPoint(controlPs.first);
        this.pushPoint(p);
        p.isControl = true;

        this.begin = this.middle;
      }
    },
    pushPoint(p) {
      //排除重复点
      if (this.line.points.length >= 1 && this.line.points[this.line.points.length - 1].x == p.x && this.line.points[this.line.points.length - 1].y == p.y)
        return;
      this.line.points.push(p);
      // console.log(this.line.points)
    },
    z_distance(b, e) {
      return Math.sqrt(Math.pow(e.x - b.x, 2) + Math.pow(e.y - b.y, 2));
    },
    computeControlPoints(k, begin, middle, end) {
      if (k > 0.5 || k <= 0)
        return;

      let diff1 = new Point(middle.x - begin.x, middle.y - begin.y)
      let diff2 = null;
      if (end)
        diff2 = new Point(end.x - middle.x, end.y - middle.y)

      // let l1 = (diff1.x ** 2 + diff1.y ** 2) ** (1 / 2)
      // let l2 = (diff2.x ** 2 + diff2.y ** 2) ** (1 / 2)

      let first = new Point(middle.x - (k * diff1.x), middle.y - (k * diff1.y))
      let second = null;
      if (diff2)
        second = new Point(middle.x + (k * diff2.x), middle.y + (k * diff2.y))
      return { first: first, second: second }
    },



    draw(isUp = false) {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);


      //绘制不包含this.line的线条
      this.pointLines.forEach((line, index) => {
        let points = line.points;
        this.ctx.fillStyle = line.lineColor;
        this.ctx.beginPath();
        this.ctx.ellipse(points[0].x - 1.5, points[0].y, 6, 3, Math.PI / 4, 0, Math.PI * 2);
        this.ctx.fill();
        this.ctx.beginPath();
        this.ctx.moveTo(points[0].x, points[0].y);
        let lastW = line.lineWidth;
        this.ctx.lineWidth = line.lineWidth;
        this.ctx.strokeStyle = line.lineColor;


        this.ctx.lineJoin = "round";
        this.ctx.lineCap = "round";
        let minLineW = line.lineWidth / 4;
        let isChangeW = false;

        let changeWidthCount = line.changeWidthCount;
        for (let i = 1; i <= points.length; i++) {
          if (i == points.length) {
            this.ctx.stroke();
            break;
          }
          if (i > points.length - changeWidthCount) {
            if (!isChangeW) {
              this.ctx.stroke();//将之前的线条不变的path绘制完
              isChangeW = true;
              if (i > 1 && points[i - 1].isControl)
                continue;
            }
            let w = (lastW - minLineW) / changeWidthCount * (points.length - i) + minLineW;
            points[i - 1].lineWidth = w;
            this.ctx.beginPath();//为了开启新的路径 否则每次stroke 都会把之前的路径在描一遍
            this.ctx.lineWidth = w;
            this.ctx.moveTo(points[i - 1].x, points[i - 1].y);//移动到之前的点
            this.ctx.lineTo(points[i].x, points[i].y);
            this.ctx.stroke();//将之前的线条不变的path绘制完
          } else {
            if (points[i].isControl && points[i + 1]) {
              this.ctx.quadraticCurveTo(points[i].x, points[i].y, points[i + 1].x, points[i + 1].y);
            } else if (i >= 1 && points[i - 1].isControl) {//上一个是控制点 当前点已经被绘制
            } else
              this.ctx.lineTo(points[i].x, points[i].y);
          }
        }
      })

      //绘制this.line线条
      if (this.line.points.length == 0) {
        return
      }
      let points;
      if (isUp)
        points = this.line.points;
      else
        points = this.line.points.clone();
      //当前绘制的线条最后几个补点 贝塞尔方式增加点
      let count = 0;
      let insertCount = 0;
      let i = points.length - 1;
      let endPoint = points[i];
      let controlPoint;
      let startPoint;
      while (i >= 0) {
        if (points[i].isControl == true) {
          controlPoint = points[i];
          count++;
        } else {
          startPoint = points[i];
        }
        if (startPoint && controlPoint && endPoint) {//使用贝塞尔计算补点
          let dis = this.z_distance(startPoint, controlPoint) + this.z_distance(controlPoint, endPoint);
          let insertPoints = this.BezierCalculate([startPoint, controlPoint, endPoint], Math.floor(dis / 6) + 1);
          insertCount += insertPoints.length;
          var index = i;//插入位置
          // 把insertPoints 变成一个适合splice的数组（包含splice前2个参数的数组） 
          insertPoints.unshift(index, 1);
          Array.prototype.splice.apply(points, insertPoints);

          //补完点后
          endPoint = startPoint;
          startPoint = null;
        }
        if (count >= 6)
          break;
        i--;
      }
      //确定最后线宽变化的点数
      let changeWidthCount = count + insertCount;
      if (isUp)
        this.line.changeWidthCount = changeWidthCount;

      //制造椭圆头
      this.ctx.fillStyle = this.line.lineColor;
      this.ctx.beginPath();
      this.ctx.ellipse(points[0].x - 1.5, points[0].y, 6, 3, Math.PI / 4, 0, Math.PI * 2);
      this.ctx.fill();

      this.ctx.beginPath();
      this.ctx.moveTo(points[0].x, points[0].y);
      let lastW = this.line.lineWidth;
      this.ctx.lineWidth = this.line.lineWidth;
      this.ctx.strokeStyle = this.line.lineColor;
      this.ctx.lineJoin = "round";
      this.ctx.lineCap = "round";
      let minLineW = this.line.lineWidth / 4;
      let isChangeW = false;
      for (let i = 1; i <= points.length; i++) {
        if (i == points.length) {
          this.ctx.stroke();
          break;
        }
        //最后的一些点线宽变细
        if (i > points.length - changeWidthCount) {
          if (!isChangeW) {
            this.ctx.stroke();//将之前的线条不变的path绘制完
            isChangeW = true;
            if (i > 1 && points[i - 1].isControl)
              continue;
          }

          //计算线宽
          let w = (lastW - minLineW) / changeWidthCount * (points.length - i) + minLineW;
          points[i - 1].lineWidth = w;
          this.ctx.beginPath();//为了开启新的路径 否则每次stroke 都会把之前的路径在描一遍
          this.ctx.lineWidth = w;
          this.ctx.moveTo(points[i - 1].x, points[i - 1].y);//移动到之前的点
          this.ctx.lineTo(points[i].x, points[i].y);
          this.ctx.stroke();//将之前的线条不变的path绘制完
        } else {
          if (points[i].isControl && points[i + 1]) {
            this.ctx.quadraticCurveTo(points[i].x, points[i].y, points[i + 1].x, points[i + 1].y);
          } else if (i >= 1 && points[i - 1].isControl) {//上一个是控制点 当前点已经被绘制
          } else
            this.ctx.lineTo(points[i].x, points[i].y);
        }
      }
    },
    BezierCalculate(poss, precision) {

      //维度，坐标轴数（二维坐标，三维坐标...）
      let dimersion = 2;

      //贝塞尔曲线控制点数（阶数）
      let number = poss.length;

      //控制点数不小于 2 ，至少为二维坐标系
      if (number < 2 || dimersion < 2)
        return null;

      let result = new Array();

      //计算杨辉三角
      let mi = new Array();
      mi[0] = mi[1] = 1;
      for (let i = 3; i <= number; i++) {

        let t = new Array();
        for (let j = 0; j < i - 1; j++) {
          t[j] = mi[j];
        }

        mi[0] = mi[i - 1] = 1;
        for (let j = 0; j < i - 2; j++) {
          mi[j + 1] = t[j] + t[j + 1];
        }
      }

      //计算坐标点
      for (let i = 0; i < precision; i++) {
        let t = i / precision;
        let p = new Point(0, 0);
        p.isAdd = true;
        result.push(p);
        for (let j = 0; j < dimersion; j++) {
          let temp = 0.0;
          for (let k = 0; k < number; k++) {
            temp += Math.pow(1 - t, number - k - 1) * (j == 0 ? poss[k].x : poss[k].y) * Math.pow(t, k) * mi[k];
          }
          j == 0 ? p.x = temp : p.y = temp;
        }
      }

      return result;
    },




    save() {
      let data = this.canvas.toDataURL("image/png"); //把canvas画板上的内容转成指定格式图片数据，并进行Base64编码
      let img = new Image();
      img.src = data;
      //   $(document.body).append(img);
    }


  }
}
</script>

<style scoped>
.handwriting {
  background-color: #dddddd;
  padding: 0px;
  display: flex;
  flex-direction: column;
  height: 100%;
}
.handwriting-header {
  height: 36px;
  background-color: #3990f1;
  display: flex;
  align-items: center;
  color: #ffffff;
  padding: 0px 12px;
  justify-content: flex-end;
}
.handwriting-img-back {
  width: 28px;
  height: 28px;
  margin-right: 12px;
}
.handwriting-img-setting {
  width: 22px;
  height: 22px;
  margin-right: 12px;
}
.handwriting-img-disable {
  opacity: 0.5;
}
.handwriting-canvas {
  background-color: #ffc6c6;
  flex-grow: 1;
}
.handwriting-setting {
  position: fixed;
  z-index: 999;
  background-color: rgba(88, 88, 88, 0.5);
  min-height: 88px;
  bottom: 0px;
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  padding: 0px 20px;
  font-size: 12px;
}
.handwriting-setting > div {
  min-width: 200px;
  display: flex;
}
.handwriting-setting-brush-container {
  display: flex;
  margin-right: 20px;
}
.handwriting-setting-brush {
  padding: 1px 12px;
  color: #222222;
  background-color: #ffffff;
  margin: 0px 6px;
  border-radius: 4px;
}
.handwriting-setting-brush-selected {
  color: #ffffff;
  background-color: #3990f1;
}
.handwriting-palette-container {
  display: flex;
}
.handwriting-palette-dot {
  width: 16px;
  height: 16px;
  border-radius: 20px;
  margin: 0px 4px;
  border: solid 2px #aaaaaa;
}
.handwriting-palette-dot-selected {
  border: solid 2px #ffffff;
}
</style>
