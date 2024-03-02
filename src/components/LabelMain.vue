<template>
  <el-container>
    <el-header>矩形标注</el-header>
    <el-main>
      <el-col :span="4">
        <div class="markdown bg-40">
          <VueMarkdown :source="markdown" :breaks="true" :typographer="true" :linkify="true" :highlight="false">
          </VueMarkdown>
        </div>
      </el-col>
      <el-col :span="16">
        <div class="image_flag bg-70">
          <div class="image bg-70">
            <div class="toolbar">
              <el-button class="tool-btn tool-btn-hand" @click="setcurrentMode(1)"
                :class="{ 'tool-btn-pressed': btn_type == 1 }"></el-button>
              <el-button class="tool-btn tool-btn-point" @click="setcurrentMode(2)"
                :class="{ 'tool-btn-pressed': btn_type == 2 }"></el-button>
              <el-button class="tool-btn tool-btn-rect" @click="setcurrentMode(3)"
                :class="{ 'tool-btn-pressed': btn_type == 3 }"></el-button>
              <el-button class="tool-btn tool-btn-polygon" @click="setcurrentMode(4)"
                :class="{ 'tool-btn-pressed': btn_type == 4 }"></el-button>
              <el-button class="tool-btn tool-btn-zoomin" @click="setcurrentMode(5)"
                :class="{ 'tool-btn-pressed': btn_type == 5 }"></el-button>
              <el-button class="tool-btn tool-btn-zoomout" @click="setcurrentMode(6)"
                :class="{ 'tool-btn-pressed': btn_type == 6 }"></el-button>
              <el-button class="tool-btn tool-btn-expand" @click="setcurrentMode(7)"
                :class="{ 'tool-btn-pressed': btn_type == 7 }"></el-button>
              <el-button class="tool-btn"></el-button>
            </div>
            <canvas id="canvas" ref="canvas"></canvas>
          </div>
          <el-carousel :interval="3000" :autoplay="false" type="card" @change="handleChange">
            <el-carousel-item v-for="item in imgList" :key="item.id">
              <h3 justify="center"><img class="carousel-image" :src="item.idView"></h3>
            </el-carousel-item>
          </el-carousel>
        </div>
      </el-col>
      <el-col :span="4">
        <div class="class_label bg-40">
          <h>类别</h>
          <ul>
            <li v-for="item in classList" v-bind:key="item"> <span class="colorbar" :style="randomRgb(item[1])"></span>{{
              item[0] }}</li>
          </ul>
          <h>标注</h>
          <ul>
            <li v-for="item in labelList" v-bind:key="item">{{ item }}</li>
          </ul>
        </div>
      </el-col>
    </el-main>
  </el-container>
</template>

<script>
import VueMarkdown from 'vue-markdown';
export default {
  name: 'LabelMain',
  components: {
    VueMarkdown
  },
  data() {
    return {
      markdown: 'Hello, **Vue Markdown**!',
      context: "",
      btn_type: 1,
      currentIndex: 0,
      imgList: [
        { id: 0, idView: require('/src/images/001.png') },
        { id: 1, idView: require('/src/images/002.png') },
        { id: 2, idView: require('/src/images/003.png') },
        { id: 3, idView: require('/src/images/004.png') },
        { id: 4, idView: require('/src/images/005.png') },
        { id: 5, idView: require('/src/images/006.png') },
        { id: 6, idView: require('/src/images/007.png') },
        { id: 7, idView: require('/src/images/008.png') },
        { id: 8, idView: require('/src/images/009.png') },
        { id: 9, idView: require('/src/images/010.png') }
      ],
      classList: [
        ["people", [50, 89, 0]],
        ["dog", [70, 0, 6]],
        ["bicycle", [70, 200, 40]],
        ["cat", [255, 60, 0]],
        ["bottle", [159, 110, 160]]
      ],
      labelList: [
        [0.1, 0.1, 0.9, 0.9, 0.9, 0.8, 0.1, 0.8],
        [0.15, 0.15, 0.8, 0.8, 0.8, 0.7, 0.15, 0.7],
        [0.15, 0.15, 0.8, 0.8, 0.8, 0.7, 0.15, 0.7],
        [0.15, 0.15, 0.8, 0.8, 0.8, 0.7, 0.15, 0.7]
      ],
      randomRgb(item) {
        let R = item[0];
        let G = item[1];
        let B = item[2];
        return {
          background: 'rgb(' + R + ',' + G + ',' + B + ', .5)'
        };
      }
    }
  },
  props: {
    msg: String
  },
  mounted() {
    var canvas = this.$refs.canvas;
    var ctx = canvas.getContext('2d');
    canvas.width = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
    let devicePixelRatio = window.devicePixelRatio || 1
    let backingStoreRatio = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1
    let ratio = devicePixelRatio / backingStoreRatio
    let canvasWidth = canvas.width
    let canvasHeight = canvas.height
    canvas.width = canvasWidth * ratio
    canvas.height = canvasHeight * ratio
    this.init_image();
  },
  methods: {
    init_image: function () {
      var canvas = this.$refs.canvas;
      var context = canvas.getContext('2d');
      var img = new Image();
      img.onload = function () {
        var w = canvas.width;
        var h = canvas.height;
        var img_w = this.width;
        var img_h = this.height;
        context.clearRect(0, 0, canvas.width, canvas.height);
        if (w / h > img_w / img_h) {
          // 等高
          const view_width = 1.0 * h * img_w / img_h;
          const offset_x = (w - view_width) / 2;
          context.drawImage(img, 0, 0, this.width, this.height, offset_x, 0, view_width, h);
        }
        else {
          //等宽
          const view_height = 1.0 * w * img_h / img_w;
          const offset_y = (h - view_height) / 2;
          context.drawImage(img, 0, 0, this.width, this.height, 0, offset_y, w, view_height);
        }
      }
      img.src = this.imgList[this.currentIndex]["idView"];
    },
    drawimage: function () {
      var canvas = this.$refs.canvas;
      var context = canvas.getContext('2d');
      var img = new Image();
      img.onload = function () {
        var w = canvas.width;
        var h = canvas.height;
        console.log(w)
        console.log(h)
        context.drawImage(img, 0, 0, this.width, this.height, 0, 0, w, h);
      }
      img.src = require('/src/images/001.png')
    },
    setcurrentMode: function (type) {
      this.btn_type = type;
    },
    getRandomColor: function () {
      return '#' + (function (color) {
        return (color += '0123456789abcdef'[Math.floor(Math.random() * 16)])
          && (color.length == 6) ? color : arguments.callee(color);
      })('');
    },
    handleChange(val) {
      this.currentIndex = val; // 更新当前图片索引值
      // this.init_image();
    }
  },
  watch:{
    currentIndex:function(){
      this.init_image();
    }
  }
}
</script>


<style scoped>
.el-container {
  height: 100%;
  padding: 0;
  margin: 0;
}

.el-header {
  padding: 0;
  height: 35px !important;
  text-align: center;
  line-height: 35px;
  color: rgb(240, 240, 240);
  background-color: rgb(75, 75, 75);
  font-size: small;
  border-bottom: 1px solid rgb(40, 40, 40);
}

.el-main {
  align-items: stretch;
  padding: 0;
  overflow: hidden;
}

.markdown {
  height: 100%;
  border-right: 1px solid rgb(40, 40, 40);
  overflow: hidden;
  color: rgb(240, 240, 240);
  padding-left: 10px;
  padding-right: 10px;
  font-size: small;
}

.VueMarkdown {
  margin: 0;
  height: 100%;
}

.image_flag {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.image {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: row;
  /* border: 1px solid white; */
}


.toolbar {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: rgb(75, 75, 75);
  width: 41px;
  padding-top: 10px;
}

.tool-btn {
  margin: 0 !important;
  padding: 0 !important;
  width: 30px;
  height: 24px;
  border-radius: 2px !important;
  border: none;
  background-color: rgb(75, 75, 75);
  background-repeat: no-repeat;
  background-position: center;
  margin-bottom: 9px !important;
}

.tool-btn:hover {
  background-color: rgb(62, 62, 62) !important;
  border: 1px solid rgb(43, 43, 43) !important;
}

.tool-btn-pressed {
  background-color: rgb(62, 62, 62) !important;
  border: 1px solid rgb(43, 43, 43) !important;
}

.tool-btn-hand {
  background-image: url('../assets/hand.png');
  /* background-color: rgb(62, 62, 62) !important;
  border: 1px solid rgb(43, 43, 43) !important; */
}

.tool-btn-point {
  background-image: url('../assets/point.png');
}

.tool-btn-rect {
  background-image: url('../assets/rect.png');
}

.tool-btn-polygon {
  background-image: url('../assets/polygon.png');
}

.tool-btn-zoomin {
  background-image: url('../assets/in.png');
}

.tool-btn-zoomout {
  background-image: url('../assets/out.png');
}

.tool-btn-expand {
  background-image: url('../assets/expand.png');
}



canvas {
  height: 100%;
  width: 100%;
  overflow: hidden;
  flex: 1;
}


.el-carousel {
  height: 100px !important;
  background-color: rgb(40, 40, 40);
}


.carousel-image {
  width: 100%;
  height: 100px !important;
  border: 1px solid rgb(220, 220, 220);
  box-sizing: border-box;
}

.el-carousel__item h3 {
  opacity: 1;
  text-align: center;
  margin-block-start: 0 !important;
  margin-block-end: 0 !important;
}

.class_label {
  height: 100%;
  border-left: 1px solid rgb(40, 40, 40);
  display: flex;
  flex-direction: column;
  color: rgb(240, 240, 240);
  font-size: small;
}

.class_label h {
  height: 10px;
  line-height: 10px;
  text-align: center;
  /* border-top: 1px solid rgb(40, 40, 40); */
  border-bottom: 1px solid rgb(40, 40, 40);
  padding: 5px;
  background-color: rgb(57, 57, 57);
}

.class_label ul {
  flex: 1;
  overflow-y: scroll;
  margin-right: -17px;
  padding-inline-start: 0px !important;
}

.class_label li {
  display: flex;
  /* border-bottom: 1px solid rgb(40, 40, 40); */
  list-style-type: none;
  height: 22px;
  vertical-align: center;
  line-height: 22px !important;
}

.class_label li:hover {
  background-color: rgb(70, 70, 70);
}

.colorbar {
  display: block;
  width: 15px;
  height: 22px;
  border: none !important;
  /* background-color: red; */
  padding: none !important;
  margin-right: 5px;
}




.grid-content {
  height: 100%;
}

.el-col {
  border-radius: 4px;
}

.bg-40 {
  background: rgb(40, 40, 40);
}

.bg-70 {
  background: rgb(70, 70, 70);
}
</style>
