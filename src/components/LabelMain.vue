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
              <el-button class="tool-btn tool-btn-hand" @click="setMode(1)"
                :class="{ 'tool-btn-pressed': btn_type == 1 }"></el-button>
              <el-button class="tool-btn tool-btn-point" @click="setMode(2)"
                :class="{'tool-btn-pressed': btn_type == 2}"></el-button>
              <el-button class="tool-btn tool-btn-rect" @click="setMode(3)" 
                :class="{'tool-btn-pressed': btn_type == 3}"></el-button>
              <el-button class="tool-btn tool-btn-polygon" @click="setMode(4)" 
                :class="{'tool-btn-pressed': btn_type == 4}"></el-button>
              <el-button class="tool-btn tool-btn-zoomin" @click="setMode(5)" 
                :class="{'tool-btn-pressed': btn_type == 5}"></el-button>
              <el-button class="tool-btn tool-btn-zoomout" @click="setMode(6)" 
                :class="{'tool-btn-pressed': btn_type == 6}"></el-button>
              <el-button class="tool-btn tool-btn-expand" @click="setMode(7)" 
                :class="{'tool-btn-pressed': btn_type == 7}"></el-button>
              <el-button class="tool-btn"></el-button>
            </div>

            <canvas id="canvas" ref="canvas" width="1928px" height="1090px"></canvas>
          </div>
          <div class="flag bg-40"></div>
        </div>
      </el-col>
      <el-col :span=" 4 ">
        <div class="labellist bg-40"></div>
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
      btn_type: 1
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
        // console.log(w);
        // console.log(h);

        context.drawImage(img, 0, 0, this.width, this.height, 0, 0, w, h);
      }
      img.src = require('/src/images/001.png')
    },
    setMode: function (type) {
      this.btn_type = type;
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
}

.el-main {
  align-items: stretch;
  padding: 0;
}

.markdown {
  height: 100%;
  border-right: 1px solid rgb(20, 20, 20);
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

.labellist {
  height: 100%;
  border-left: 1px solid rgb(20, 20, 20);
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
