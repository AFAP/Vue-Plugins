<template>
  <div class="handwriting">
    <canvas id="canvas-handwriting" class="handwriting-canvas"></canvas>

    <!-- <div>
      X:{{ currentPoint.x }}
    </div>
    <div>
      Y:{{ currentPoint.y }}
    </div> -->

  </div>
</template>

<script>
export default {
  name: 'HandWriting',
  props: {
    msg: String,

  },
  data() {
    return {
      canvas: null,
      board: null,
      showPalette: false,
      showBrush: false,
      mousePress: false,
      last: null,
      currentPoint: {}
    };
  },
  mounted() {
    this.canvas = document.getElementById('canvas-handwriting');
    this.board = this.canvas.getContext('2d');
    this.board.lineWidth = "12"; //设置画笔粗细
    this.board.strokeStyle = "#000";
    this.board.lineJoin = "round"; //设置画笔轨迹基于圆点拼接


    let width = document.documentElement["clientWidth"];
    let height = document.documentElement["clientHeight"];

    if (window.devicePixelRatio) {
      this.canvas.style.width = width + "px";
      this.canvas.style.height = height + "px";
      this.canvas.height = height * window.devicePixelRatio;
      this.canvas.width = width * window.devicePixelRatio;
      this.board.scale(window.devicePixelRatio, window.devicePixelRatio);
    }



    this.canvas.onmousedown = this.beginDraw;
    this.canvas.onmousemove = this.drawing;
    this.canvas.onmouseup = this.endDraw;

    this.canvas.addEventListener("touchstart", this.beginDraw);
    this.canvas.addEventListener("touchmove", this.drawing);
    this.canvas.addEventListener("touchend", this.endDraw);


  },
  methods: {
    isPaused() {
      return this.showBrush || this.showPalette;
    },
    beginDraw(event) {
      console.log('beginDraw')
      this.mousePress = true;
    },
    endDraw(event) {
      console.log('endDraw')
      this.mousePress = false;
      event.preventDefault();
      this.last = null;
    },
    drawing(event) {
      event.preventDefault();
      if (!this.mousePress) {
        return;
      }
      // console.log('this.last', this.last)
      let xy = this.GetPos(event);
      if (this.last != null) {
        this.board.beginPath();
        this.board.moveTo(this.last.x, this.last.y);
        this.board.lineTo(xy.x, xy.y);
        this.board.stroke();
      }
      // console.log('drawing------w3333333333------------', xy)
      this.last = xy;
      this.currentPoint = xy;
    },
    GetPos(event) {
      // console.log(event)
      let x = 0, y = 0;
      if (this.isTouchEvent(event)) {
        let touch = event.targetTouches[0];
        x = touch.clientX - touch.target.offsetLeft;
        y = touch.clientY - touch.target.offsetTop;
      } else {
        x = event.offsetX;
        y = event.offsetY;
      }

      return { x, y };
    },
    isTouchEvent(event) {
      return event.type.indexOf('touch') >= 0;
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
}
.handwriting-canvas {
  background-color: #ffc6c6;
}
</style>
