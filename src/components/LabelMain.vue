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

            <canvas id="canvas" ref="canvas" width="1928px" height="1090px"></canvas>
          </div>
          <div class="flag bg-40">
            <el-carousel :interval="4000" type="card" height="70px">
              <el-carousel-item v-for="item in imgList" :key="item.id">
                <h3> <img :src="item.idView" class="banner"></h3>
              </el-carousel-item>
            </el-carousel>
          </div>
        </div>
      </el-col>
      <el-col :span="4">
        <div class="class_label bg-40">
          <h>类别</h>
          <ul>
            <li v-for="item in classList" v-bind:key="item"> <span class="colorbar"
              :style="randomRgb(item[1])"></span>{{ item[0] }}</li>
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
        ["bottle", [255, 10, 60]]
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

    this.drawimage();
  },
  methods: {
    drawimage: function () {
      var canvas = this.$refs.canvas;
      var context = canvas.getContext('2d');
      var img = new Image();

      img.onload = function () {
        canvas.height = this.height;
        canvas.width = this.width;

        var w = canvas.width;
        var h = canvas.height;
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
    }
  }
}
</script>


<style scoped>
.el-container {
  height: 100vh;
  padding: 0;
}

.el-header {
  padding: 0;
  height: 35px !important;
  text-align: center;
  line-height: 35px;
  color: rgb(240, 240, 240);
  background-color: rgb(20, 20, 20);
  font-size: small;
  border-bottom: 1px solid rgb(40, 40, 40);
}

.el-main {
  align-items: stretch;
  padding: 0;
}

.markdown {
  height: 100%;
  border-right: 1px solid rgb(35, 35, 35);
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
}


.toolbar {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: rgb(70, 70, 70);
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
  background-color: rgb(70, 70, 70);
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

.flag {
  height: 100px;
  border-top: 1px solid rgb(20, 20, 20);
}

.banner {
  display: block;
  width: 100%;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.class_label {
  height: 100%;
  border-left: 1px solid rgb(35, 35, 35);
  display: flex;
  flex-direction: column;
  color: rgb(240, 240, 240);
  font-size: small;
}

.class_label h {
  height: 10px;
  text-align: center;
}

.class_label ul {
  flex: 1;
  overflow-y: scroll;
  margin-right: -17px;
  padding-inline-start: 0px !important;
}

.class_label li {
  display: flex;
  /* border-bottom: 1px solid rgb(35, 35, 35); */
  list-style-type: none;
  height: 22px;
  vertical-align: center;
  line-height: 22px !important;
}
.class_label li :hover{
  background-color: rgb(70,70,.70);
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
