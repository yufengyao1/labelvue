<template>
  <el-container>
    <el-header>图像标注</el-header>
    <div class="main-container">
      <div class="markdown" v-show="showLeftPanel">
        <div class="title-left">
          <div class="panel-zhanwei"></div><span>菜单</span>
          <el-button class="fold-left" @click="showLeftPanel = false"></el-button>
        </div>
        <div class="markdown-container">
          <VueMarkdown :source="markdown" :breaks="true" :typographer="true" :linkify="true" :highlight="false">
          </VueMarkdown>
        </div>
      </div>

      <div class="image_flag">
        <div class="toolbar-canvas">
          <div class="toolbar">
            <el-button class="tool-btn fold-right" @click="showLeftPanel = true" v-show="!showLeftPanel"></el-button>
            <el-button class="tool-btn tool-btn-hand" @click="setcurrentMode(1)"
              :class="{ 'tool-btn-pressed': btn_type == 1 }"></el-button>

            <el-button class="tool-btn tool-btn-point" @click="drawPoint"
              :class="{ 'tool-btn-pressed': btn_type == 2 }"></el-button>
            <el-button class="tool-btn tool-btn-rect" @click="drawRect"
              :class="{ 'tool-btn-pressed': btn_type == 3 }"></el-button>
            <el-button class="tool-btn tool-btn-polygon" @click="drawPolygon"
              :class="{ 'tool-btn-pressed': btn_type == 4 }"></el-button>

            <el-button class="tool-btn tool-btn-modify" @click="modifyPoint"
              :class="{ 'tool-btn-pressed': btn_type == 8 }"></el-button>
            <el-button class="tool-btn tool-btn-zoomin" @click="zoomIn"
              :class="{ 'tool-btn-pressed': btn_type == 5 }"></el-button>
            <el-button class="tool-btn tool-btn-zoomout" @click="zoomOut"
              :class="{ 'tool-btn-pressed': btn_type == 6 }"></el-button>
            <el-button class="tool-btn tool-btn-expand" @click="expandImg"
              :class="{ 'tool-btn-pressed': btn_type == 7 }"></el-button>
            <el-color-picker v-model="color_init" size="small"></el-color-picker>


          </div>
          <div class="center-container">
            <div class="title-center">{{ imgFile }} @ {{ currentJindu }}</div>
            <div class="canvas-container" ref="canvas_container">
              <div class="canvas1-container">
                <canvas id="canvas" ref="canvas" @mousedown="handleMouseDown" @mouseup="handleMouseUp"
                  @mousemove="handleMouseMove" @mouseout="handleMouseOut" @dblclick="handleDlbClick"></canvas>

              </div>
              <div class="canvas2-container" ref="canvas_container_small"
                :style="{ width: divWidth + 'px', height: divHeight + 'px' }">
                <canvas id="canvas-small" ref="canvas_small" @mousedown="handleSmallMouseDown"></canvas>
              </div>
            </div>
          </div>

          <el-dialog title="选择类别" :visible.sync="dialogVisible" width="30%">
            <div class="dialog-container">
              <el-select v-model="selected_class_name" placeholder="请选择" size="small">
                <el-option v-for="item in classList" :key="item.id" :label="item.name" :value="item.name">
                </el-option>
              </el-select>
              <div class="corlor_class_container">
                <div class="color_class" v-for="item in classList" :key="item.id"
                  :class="{ 'color_class_selected': item.name == selected_class_name }"
                  @click="selected_class_name = item.name" :style="{ backgroundColor: item.color }"><span>{{ item.name
                    }}</span></div>
              </div>
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button type="primary" @click="handleClose(0)" size="small">取 消</el-button>
              <el-button type="primary" @click="handleClose(1)" size="small">确 定</el-button>
            </span>
          </el-dialog>
        </div>

        <div class="carousel-container">
          <button class="prev" @click="setLastPage">
            <i class="el-icon-arrow-left"></i>
          </button>
          <div class="imgdet" v-for="item, index in dataListOnePage" :key="item.id" @click="setCurrentImg(index)">
            <img :src="item.image" class="carousel-image"
              :class="{ 'carousel-image-selected': currentImgIndex == index }" />
          </div>
          <button class="next" @click="setNextPage">
            <i class="el-icon-arrow-right"></i>
          </button>

        </div>
      </div>

      <div class="class_label">
        <span class="title">类别</span>
        <ul>
          <li v-for="item in classList" :key="item.id"> <span class="colorbar"
              :style="{ background: item.color }"></span><span class="label-name">{{ item.name }}</span><span
              class="label-remove" @click="removeClass(item.id)">移除</span></li>
        </ul>
        <input v-model="input_class" placeholder="输入类别回车以添加" class="add-class-input" @keyup.enter="addClass" />
        <span class="title">标注</span>
        <ul class="labels-ul">
          <li class="label-some" v-for="item in labelList2" :key="item.id" @click="zoomToLabel(item.id)"> <span
              :class="getLabelClass(item.class_type)"></span> <span class="label-classname">{{ item.class_name
              }}</span><span class="label-remove" @click="removeLabel(item.id)">移除</span>
          </li>
        </ul>
      </div>
    </div>
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
      imgList: [ //图像列表
        { id: 0, image: require('/src/images/001.png'), file: '/src/images/001.png' },
        { id: 1, image: require('/src/images/002.png'), file: '/src/images/002.png' },
        { id: 2, image: require('/src/images/003.png'), file: '/src/images/003.png' },
        { id: 3, image: require('/src/images/004.png'), file: '/src/images/004.png' },
        { id: 4, image: require('/src/images/005.png'), file: '/src/images/005.png' },
        { id: 5, image: require('/src/images/006.png'), file: '/src/images/006.png' },
        { id: 6, image: require('/src/images/007.png'), file: '/src/images/007.png' },
        { id: 7, image: require('/src/images/008.png'), file: '/src/images/008.png' },
        { id: 8, image: require('/src/images/009.png'), file: '/src/images/009.png' },
        { id: 9, image: require('/src/images/011.png'), file: '/src/images/011.png' }
      ],
      classList: [ //类列表
        { "id": 0, "name": "default", "color": this.color16() },
        { "id": 1, "name": "people", "color": this.color16() },
        { "id": 2, "name": "dog", "color": this.color16() },
        { "id": 3, "name": "bicycle", "color": this.color16() },
        { "id": 4, "name": "cat", "color": this.color16() },
        { "id": 5, "name": "horse", "color": this.color16() },
        { "id": 6, "name": "monkey", "color": this.color16() },
        { "id": 7, "name": "tiger", "color": this.color16() },
        { "id": 8, "name": "house", "color": this.color16() },
        { "id": 9, "name": "table", "color": this.color16() },
        { "id": 10, "name": "chair", "color": this.color16() },
        { "id": 11, "name": "point", "color": this.color16() }
      ],
      allLabelList: [], //所有已完成的标注
      currentLabelList: { "points": [], "rects": [], "polys": [] }, //当前页所有标注 { "points": [], "rects": [], "polys": [{"class":"dog","points":[]}] },
      indexInAllLabel: 0, //当前图对应的label列表中的序号
      markdown: 'Hello, *Vue Markdown!*',
      showLeftPanel: true,
      btn_type: 1,
      currentImgIndex: 0, //当前图片index
      currentImgPage: 0, //当前页面
      imgsOnePage: 5,//每页显示的图片数量
      currentImg: null,
      offset_x: 0,
      offset_y: 0,
      real_width: 0, //前景宽，非画布宽
      real_height: 0, //前景高
      currentScaleSize: 1,
      currentHighlightId: -1, //当前需要高亮
      dragging: false,
      startX: 0, //鼠标点击的开始点
      startY: 0, //鼠标点击的开始点
      canvas: null,
      canvas_container: null,
      canvas_small: null,
      canvas_ratio: 2,
      color_init: "#50E41A",
      color_map: { "people": "#ff0000", "dog": "#ff0000" },
      current_poly_points: [], //当前绘制的点
      selected_class_name: "default", //dialog选中的类别
      dialogVisible: false,
      current_img_index: 0, //当前图片index
      input_class: "", //添加类名
      imgFile: "", //当前图片路径
      divWidth: 0,
      divHeight: 0,
      mouseXRate: 0, //当前鼠标点位置，百分比
      mouseYRate: 0, //当前鼠标点位置，百分比
      selectedPoint: [] //修改模式选中的点
    }
  },
  computed: {
    dataListOnePage: function () {
      var tmp = []
      for (var i = 0, len = this.imgList.length; i < len; i += this.imgsOnePage) {
        tmp.push(this.imgList.slice(i, i + this.imgsOnePage))
      }
      return tmp[this.currentImgPage]
    },
    classList2: function () {
      var res = {};
      for (let i = 0; i < this.classList.length; i++) {
        let class_name = this.classList[i].name;
        res[class_name] = this.classList[i].color;
      }
      return res
    },
    labelList2: function () {
      var res = [];
      if (this.currentLabelList["points"].length > 0) {
        res.push({ "id": 10000, "class_name": "point", "color": this.classList2["point"], "class_type": "point" });
      }
      for (let i = 0; i < this.currentLabelList["rects"].length; i++) {
        let class_name = this.currentLabelList["rects"][i]["class_name"];
        res.push({ "id": this.currentLabelList["rects"][i]["id"], "class_name": class_name, "color": this.classList2[class_name], "class_type": "rect" });
      }
      for (let i = 0; i < this.currentLabelList["polys"].length; i++) {
        let class_name = this.currentLabelList["polys"][i]["class_name"];
        res.push({ "id": this.currentLabelList["polys"][i]["id"], "class_name": class_name, "color": this.classList2[class_name], "class_type": "poly" });
      }
      return res;
    },
    currentJindu: function () {
      return (this.currentImgIndex + this.currentImgPage * this.imgsOnePage + 1).toString() + "/" + this.imgList.length.toString();
    },
    canvasRect: function () {
      var x1, y1, x2, y2 = 0;
      if (this.offset_x > 0) {
        x1 = 0;
      }
      else {
        x1 = 1.0 * Math.abs(this.offset_x) / this.real_width;
      }

      if (this.offset_y > 0) {
        y1 = 0;
      }
      else {
        y1 = 1.0 * Math.abs(this.offset_y) / this.real_height;
      }

      if (this.offset_x + this.real_width <= this.canvas.width) {
        x2 = 1
      }
      else {
        x2 = (this.canvas.width - this.offset_x) / this.real_width;
      }

      if (this.offset_y + this.real_height <= this.canvas.height) {
        y2 = 1;
      }
      else {
        y2 = (this.canvas.height - this.offset_y) / this.real_height;
      }
      return [x1, y1, x2, y2];
    }
  },
  props: {
    msg: String
  },
  mounted() {
    this.justifyCanvas();
    this.init_image();
    this.startResizeObserver();
  },
  beforeDestroy() {
    this.stopResizeObserver();
  },
  methods: {
    startResizeObserver() {
      const resizeBox = this.$refs.canvas_container;
      const resizeObserver = new ResizeObserver(entries => {
        this.justifyCanvas(); //重新调整canvas大小
        this.expandImg();
        this.drawCanvasRect();
      });
      resizeObserver.observe(resizeBox);
      this.resizeObserver = resizeObserver;
    },
    stopResizeObserver() {
      if (this.resizeObserver) {
        this.resizeObserver.disconnect();
      }
    },
    justifyCanvas: function () {
      this.canvas_container = this.$refs.canvas_container;
      this.canvas = this.$refs.canvas;
      var ctx = this.canvas.getContext('2d');

      let devicePixelRatio = window.devicePixelRatio || 1
      let backingStoreRatio = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1
      this.canvas_ratio = devicePixelRatio / backingStoreRatio


      this.canvas.width = this.canvas.offsetWidth * this.canvas_ratio
      this.canvas.height = this.canvas.offsetHeight * this.canvas_ratio

      this.canvas_small = this.$refs.canvas_small;
      this.canvas_small.width = this.canvas_small.offsetWidth * this.canvas_ratio
      this.canvas_small.height = this.canvas_small.offsetHeight * this.canvas_ratio

      // this.drawImageAndPoints();
    },
    init_image: function () {
      this.imgFile = this.dataListOnePage[this.currentImgIndex]["file"]
      var label_exist = false;
      for (let i = 0; i < this.allLabelList.length; i++) {
        if (this.allLabelList[i]["file"] == this.imgFile) {
          this.indexInAllLabel = i; //记录序号，用于后续更新原labellist
          this.currentLabelList = this.allLabelList[i];
          label_exist = true;
          break;
        }
      }
      if (!label_exist) {
        this.currentLabelList = { "points": [], "rects": [], "polys": [], "file": this.imgFile };
        this.allLabelList.push(this.currentLabelList);
        this.indexInAllLabel = this.allLabelList.length - 1;
      }

      var img = new Image();
      var self = this;
      self.real_height = 0;
      self.real_width = 0;
      self.offset_x = 0;
      self.offset_y = 0;
      img.onload = function () {
        self.currentImg = img;
        self.expandImg();

        self.init_small_img(); //设置缩略图
        self.drawCanvasRect();
      }
      img.src = this.dataListOnePage[this.currentImgIndex]["image"];

    },
    init_small_img: function () {
      this.canvas_small = this.$refs.canvas_small;
      var context = this.canvas_small.getContext('2d');

      var img_w = this.currentImg.width;
      var img_h = this.currentImg.height;

      var css_h = this.canvas.offsetHeight / 4;
      var css_w = this.canvas.offsetWidth / 4;

      this.canvas_small.height = this.canvas.height / 4;
      this.canvas_small.width = this.canvas.width / 4;

      if (img_w >= img_h) {
        this.canvas_small.height = 1.0 * this.canvas_small.width * img_h / img_w;
        css_h = 1.0 * css_w * img_h / img_w;
      }
      else {
        this.canvas_small.width = this.canvas_small.height * img_w / img_h;
        css_w = css_h * img_w / img_h;
      }
      // console.log([this.canvas_small.width, this.canvas_small.height]);
      // console.log([css_w, css_h]);
      this.divHeight = css_h;
      this.divWidth = css_w;
      context.clearRect(0, 0, this.canvas_small.width, this.canvas_small.height);
      context.drawImage(this.currentImg, 0, 0, this.currentImg.width, this.currentImg.height, 0, 0, this.canvas_small.width, this.canvas_small.height);
    },
    drawCanvasRect: function () {
      var context = this.canvas_small.getContext('2d');
      var x1 = this.canvasRect[0] * this.canvas_small.width;
      var y1 = this.canvasRect[1] * this.canvas_small.height;
      var x2 = this.canvasRect[2] * this.canvas_small.width;
      var y2 = this.canvasRect[3] * this.canvas_small.height;
      context.lineWidth = 2
      context.clearRect(0, 0, this.canvas_small.width, this.canvas_small.height);
      context.drawImage(this.currentImg, 0, 0, this.currentImg.width, this.currentImg.height, 0, 0, this.canvas_small.width, this.canvas_small.height);
      context.strokeStyle = "#ff0000";

      context.fillStyle = "rgba(255,255,255,0.5)";

      context.beginPath(); // 开始画图
      context.strokeRect(x1, y1, x2 - x1, y2 - y1)
      context.fillRect(x1, y1, x2 - x1, y2 - y1)
      context.closePath();

      context.lineWidth = 2
      context.beginPath(); // 开始画图
      context.moveTo(0, (y1 + y2) / 2);
      context.lineTo(this.canvas_small.width, (y1 + y2) / 2);
      context.stroke()

      context.beginPath(); // 开始画图
      context.moveTo((x1 + x2) / 2, 0);
      context.lineTo((x1 + x2) / 2, this.canvas_small.height);
      context.stroke()
    },
    setLastPage: function () {
      if (this.currentImgPage > 0) {
        this.currentImgPage -= 1;
      }
      this.setCurrentImg(this.imgsOnePage - 1);
      this.init_image();
    },
    setNextPage: function () {
      if (this.currentImgPage < this.imgList.length / this.imgsOnePage - 1) {
        this.currentImgPage += 1;
        this.setCurrentImg(0);
        this.init_image();
      }

    },
    setCurrentImg(index) {
      this.currentImgIndex = index;
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
    getLabelClass(class_type) {
      if (class_type == "point") {
        return "label-class-point";
      }
      else if (class_type == "rect") {
        return "label-class-rect";
      }
      else if (class_type == "poly") {
        return "label-class-poly";
      }
    },
    handleChange: function (val) {
      this.currentImgIndex = val; // 更新当前图片索引值
    },
    drawPoint: function () {
      this.setcurrentMode(2);//draw rect
    },
    modifyPoint: function () {
      this.setcurrentMode(8);//edit
    },
    drawRect: function () {
      this.setcurrentMode(3);//draw rect
    },
    drawPolygon: function () {
      this.setcurrentMode(4);//draw polygon
    },
    zoomIn: function () {
      this.setcurrentMode(1);//放大
      if (this.currentScaleSize > 15) return;
      const current_real_height = this.real_height + parseInt(this.real_height / 3);
      const current_real_width = this.real_width + parseInt(this.real_width / 3);
      const offset_x = parseInt((current_real_width - this.real_width) / 2);
      const offset_y = parseInt((current_real_height - this.real_height) / 2);
      this.offset_x -= offset_x;
      this.offset_y -= offset_y;
      this.real_height = parseInt(current_real_height);
      this.real_width = parseInt(current_real_width);
      this.drawImageAndPoints();
      this.drawCanvasRect();
      this.currentScaleSize += 1;
    },
    zoomOut: function () {
      this.setcurrentMode(1);//缩小
      const current_real_height = this.real_height - parseInt(this.real_height / 4);
      const current_real_width = this.real_width - parseInt(this.real_width / 4);
      const offset_x = parseInt((this.real_width - current_real_width) / 2);
      const offset_y = parseInt((this.real_height - current_real_height) / 2);
      this.offset_x += offset_x;
      this.offset_y += offset_y;
      this.real_height = parseInt(current_real_height);
      this.real_width = parseInt(current_real_width);
      this.drawImageAndPoints();
      this.drawCanvasRect();
      this.currentScaleSize -= 1;
    },
    expandImg: function () {
      var canvas_w = this.canvas.width;
      var canvas_h = this.canvas.height;
      var img_w = this.currentImg.width;
      var img_h = this.currentImg.height;
      if (canvas_w / canvas_h > img_w / img_h) {
        // 等高
        const view_width = 1.0 * canvas_h * img_w / img_h;
        const offset_x = (canvas_w - view_width) / 2;
        this.offset_x = offset_x;
        this.offset_y = 0;
        this.real_width = view_width;
        this.real_height = canvas_h;
      }
      else {
        //等宽
        const view_height = 1.0 * canvas_w * img_h / img_w;
        const offset_y = (canvas_h - view_height) / 2;
        this.offset_x = 0;
        this.offset_y = offset_y;
        this.real_height = view_height;
        this.real_width = canvas_w;
      }
      this.drawImageAndPoints();
      this.drawCanvasRect();
    },
    handleMouseDown: function (e) {
      this.dragging = true;
      this.currentHighlightId = -1; //取消高亮
      this.startX = e.clientX - this.canvas_container.offsetLeft + window.pageXOffset;
      this.startY = e.clientY - this.canvas_container.offsetTop + window.pageYOffset;
      var mouseX, mouseY, x_rate, y_rate = 0;
      switch (this.btn_type) {
        case 3: //rect
          if (this.current_poly_points.length == 0) {
            mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio - this.offset_x;
            mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio - this.offset_y;
            x_rate = mouseX / this.real_width;
            y_rate = mouseY / this.real_height;
            this.current_poly_points.push([x_rate, y_rate]); //保存当前多边形点}
          }
          break;
        case 4: //polygon
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio - this.offset_x;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio - this.offset_y;
          x_rate = mouseX / this.real_width;
          y_rate = mouseY / this.real_height;
          this.current_poly_points.push([x_rate, y_rate]); //保存当前多边形点}
          break;
      }
    },
    handleSmallMouseDown: function (e) {
      var canvas_container_small = this.$refs.canvas_container_small;
      var x = e.clientX - canvas_container_small.offsetLeft + window.pageXOffset - this.canvas_container.offsetLeft;
      var y = e.clientY - canvas_container_small.offsetTop + window.pageYOffset - this.canvas_container.offsetTop;
      var x_ratio = 1.0 * x / this.canvas_small.offsetWidth;
      var y_ratio = 1.0 * y / this.canvas_small.offsetHeight;

      this.offset_x = this.canvas.offsetWidth - x_ratio * this.real_width;
      this.offset_y = this.canvas.offsetHeight - y_ratio * this.real_height;


      this.drawImageAndPoints();
      this.drawCanvasRect();
    },
    handleMouseMove: function (e) {
      var mouseX, mouseY = 0;
      var context;
      switch (this.btn_type) {
        case 1: //平移
          if (this.dragging) {
            mouseX = e.clientX - this.canvas_container.offsetLeft + window.pageXOffset;
            mouseY = e.clientY - this.canvas_container.offsetTop + window.pageYOffset;
            const dx = mouseX - this.startX;
            const dy = mouseY - this.startY;
            this.offset_x += dx * this.canvas_ratio;
            this.offset_y += dy * this.canvas_ratio;
            this.startX = mouseX;
            this.startY = mouseY;
            context = this.canvas.getContext('2d');
            context.clearRect(0, 0, this.canvas.width, this.canvas.height);
            context.drawImage(this.currentImg, 0, 0, this.currentImg.width, this.currentImg.height, this.offset_x, this.offset_y, this.real_width, this.real_height);
            this.drawImageAndPoints();
            this.drawCanvasRect();
          }
          break;
        case 2: //点
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio;
          // console.log([e.clientX, this.canvas_container.offsetLeft, window.pageXOffset])
          // console.log([e.clientY,this.canvas.offsetRight,window.pageYOffset])
          this.drawImageAndPoints(); //绘制历史点
          //绘制当前移动的点
          context = this.canvas.getContext('2d');
          context.beginPath(); // 开始画图
          context.arc(mouseX, mouseY, 5, 0, 360); //（圆心横纵坐标，圆心纵坐标，半径，0，360） |0，360表示画出的是一个圆
          context.fillStyle = this.color_init; //图形填充颜色
          context.fill(); //画实心圆
          context.closePath();

          break;
        case 3: //rect
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio;
          //绘制当前移动的点
          if (this.current_poly_points.length > 0) {
            this.drawImageAndPoints(); //绘制历史点
            context = this.canvas.getContext('2d');
            const x1 = this.current_poly_points[0][0] * this.real_width + this.offset_x;
            const y1 = this.current_poly_points[0][1] * this.real_height + this.offset_y;
            const x2 = mouseX
            const y2 = mouseY;

            context.beginPath(); // 开始画图
            context.strokeStyle = this.color_init;
            context.lineWidth = 3; // 设置边框宽度为5像素
            context.strokeRect(x1, y1, x2 - x1, y2 - y1)
            context.closePath();
          }
          break;
        case 4: //polygon
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio;
          var start_x, start_y = 0;
          //绘制当前移动的点
          if (this.current_poly_points.length > 0) {
            this.drawImageAndPoints(); //绘制历史点
            context = this.canvas.getContext('2d');
            context.beginPath()
            for (let i = 0; i < this.current_poly_points.length; i++) {
              let p = this.current_poly_points[i];
              let cx = p[0] * this.real_width + this.offset_x;
              let cy = p[1] * this.real_height + this.offset_y;
              if (i === 0) {
                context.moveTo(cx, cy);
                start_x = cx;
                start_y = cy;
              }
              context.lineTo(cx, cy);
              // 在点处画一个小矩形
              // context.fillRect(cx - 5, cy - 5, 10, 10)
            }
            context.lineTo(mouseX, mouseY);
            context.lineTo(start_x, start_y);
            context.stroke()
          }
          break;
        case 8: //修改点
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio;
          this.mouseXRate = mouseX / this.real_width;
          this.mouseYRate = mouseY / this.real_height;

          if (!this.dragging) { //没有拖拽
            this.selectedPoint = [];
            var distance = 20;
            var tmp_x = this.mouseXRate * this.real_width;//+this.offset_x;
            var tmp_y = this.mouseYRate * this.real_height;//+this.offset_y;
            //查找点
            for (let i = 0; i < this.currentLabelList["points"].length; i++) {
              var x = this.currentLabelList["points"][i][0] * this.real_width + this.offset_x;
              var y = this.currentLabelList["points"][i][1] * this.real_height + this.offset_y;
              if (Math.abs(x - tmp_x) < distance && Math.abs(y - tmp_y) < distance) {
                this.selectedPoint = ["points", i];
              }
            }
            //查找矩形
            for (let i = 0; i < this.currentLabelList["rects"].length; i++) {
              let rect = this.currentLabelList["rects"][i]["points"];
              let x1 = rect[0][0] * this.real_width + this.offset_x;
              let y1 = rect[0][1] * this.real_height + this.offset_y;
              let x2 = rect[1][0] * this.real_width + this.offset_x;
              let y2 = rect[1][1] * this.real_height + this.offset_y;
              if (Math.abs(x1 - tmp_x) < distance && Math.abs(y1 - tmp_y) < distance) {
                this.selectedPoint = ["rects", i, 0]
                // console.log(this.selectedPoint);
              }
              if (Math.abs(x2 - tmp_x) < distance && Math.abs(y2 - tmp_y) < distance) {
                this.selectedPoint = ["rects", i, 1]
                // console.log(this.selectedPoint);
              }
            }
            //查找多边形     
            for (let i = 0; i < this.currentLabelList["polys"].length; i++) {
              var points = this.currentLabelList["polys"][i]["points"];
              for (let j = 0; j < points.length; j++) {
                let p = points[j];
                let cx = p[0] * this.real_width + this.offset_x;
                let cy = p[1] * this.real_height + this.offset_y;
                if (Math.abs(cx - tmp_x) < distance && Math.abs(cy - tmp_y) < distance) {
                  this.selectedPoint = ["polys", i, j]
                }
              }
            }
            this.drawImageAndPoints();
          }
          else { //拖拽
            if (this.selectedPoint.length > 0) {
              mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio - this.offset_x;
              mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio - this.offset_y;
              var x_rate = mouseX / this.real_width;
              var y_rate = mouseY / this.real_height;

              var type = this.selectedPoint[0];
              if (this.selectedPoint.length == 2) {
                let index = this.selectedPoint[1];
                this.currentLabelList[type][index][0] = x_rate;
                this.currentLabelList[type][index][1] = y_rate;
              }
              else {
                let index1 = this.selectedPoint[1];
                let index2 = this.selectedPoint[2];
                this.currentLabelList[type][index1]["points"][index2][0] = x_rate;
                this.currentLabelList[type][index1]["points"][index2][1] = y_rate;
              }
            }
            this.drawImageAndPoints();
          }

          break;
      }
    },
    handleMouseUp: function (e) {
      this.dragging = false;
      this.currentHighlightId = -1; //取消高亮
      var mouseX, mouseY, x_rate, y_rate = 0
      switch (this.btn_type) {
        case 2: //point
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio - this.offset_x;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio - this.offset_y;
          x_rate = mouseX / this.real_width;
          y_rate = mouseY / this.real_height;
          this.currentLabelList["points"].push([x_rate, y_rate]);
          break;
        case 3: //rect
          mouseX = (e.clientX - this.canvas_container.offsetLeft + window.pageXOffset) * this.canvas_ratio - this.offset_x;
          mouseY = (e.clientY - this.canvas_container.offsetTop + window.pageYOffset) * this.canvas_ratio - this.offset_y;
          x_rate = mouseX / this.real_width;
          y_rate = mouseY / this.real_height;

          if (Math.abs(mouseX - this.current_poly_points[0][0] * this.real_width) > 5 || Math.abs(mouseY - this.current_poly_points[0][1] * this.real_height) > 5) {
            this.current_poly_points.push([x_rate, y_rate]); //保存当前多边形点
            if (this.current_poly_points.length == 2) { //已经有两个点，弹出标注类别确认框
              this.dialogVisible = true;
            }
          }
          break;

      }
    },
    handleDlbClick: function () {
      switch (this.btn_type) {
        case 4: //polygon
          this.dialogVisible = true;
          break
      }
    },
    handleMouseOut: function () {
      this.drawImageAndPoints();
    },
    drawImageAndPoints: function () { //绘制历史点
      //绘制点
      var context = this.canvas.getContext('2d');
      context.clearRect(0, 0, this.canvas.width, this.canvas.height);
      context.drawImage(this.currentImg, 0, 0, this.currentImg.width, this.currentImg.height, this.offset_x, this.offset_y, this.real_width, this.real_height);

      var fill_style = null;
      if ("point" in this.classList2) {
        fill_style = this.classList2["point"]; //图形填充颜色
      }
      else {
        fill_style = this.color_init; //图形填充颜色
      }
      if (this.currentHighlightId == 10000) {
        fill_style = "#ff0000"; //图形填充颜色
      }

      for (let i = 0; i < this.currentLabelList["points"].length; i++) {
        context.fillStyle = fill_style
        context.beginPath(); // 开始画图
        var x = this.currentLabelList["points"][i][0] * this.real_width + this.offset_x;
        var y = this.currentLabelList["points"][i][1] * this.real_height + this.offset_y;
        context.arc(x, y, 7, 0, 360);
        context.fill(); //画实心圆 
        context.closePath();
      }
      //绘制矩形
      context.lineWidth = 3; // 设置边框宽度为5像素
      for (let i = 0; i < this.currentLabelList["rects"].length; i++) {
        let rect = this.currentLabelList["rects"][i]["points"];
        let class_name = this.currentLabelList["rects"][i]["class_name"];
        if (class_name in this.classList2) {
          fill_style = this.classList2[class_name]; //图形填充颜色
        }
        else {
          fill_style = this.color_init; //图形填充颜色
        }

        context.strokeStyle = fill_style;
        context.fillStyle = fill_style;
        if (this.currentHighlightId == this.currentLabelList["rects"][i]["id"]) {
          context.strokeStyle = "#ff0000"; //图形填充颜色
          context.fillStyle = "#ff0000"; //图形填充颜色
        }

        let x1 = rect[0][0] * this.real_width + this.offset_x;
        let y1 = rect[0][1] * this.real_height + this.offset_y;
        let x2 = rect[1][0] * this.real_width + this.offset_x;
        let y2 = rect[1][1] * this.real_height + this.offset_y;


        context.beginPath(); // 开始画图
        context.strokeRect(x1, y1, x2 - x1, y2 - y1)
        context.closePath();

        context.beginPath();
        context.arc(x1, y1, 7, 0, 360);
        context.fill(); //画实心圆 
        context.closePath();

        context.beginPath();
        context.arc(x2, y2, 7, 0, 360);
        context.fill(); //画实心圆 
        context.closePath();
      }

      //绘制多边形     
      var start_x, start_y = 0;
      for (let i = 0; i < this.currentLabelList["polys"].length; i++) {
        let points = this.currentLabelList["polys"][i]["points"];
        let class_name = this.currentLabelList["polys"][i]["class_name"];
        if (class_name in this.classList2) {
          fill_style = this.classList2[class_name]; //图形填充颜色
        }
        else {
          fill_style = this.color_init; //图形填充颜色
        }

        context.strokeStyle = fill_style;
        context.fillStyle = fill_style;
        if (this.currentHighlightId == this.currentLabelList["polys"][i]["id"]) {
          context.strokeStyle = "#ff0000"; //图形填充颜色
          context.fillStyle = "#ff0000"; //图形填充颜色
        }
        context.beginPath()
        for (let i = 0; i < points.length; i++) {
          let p = points[i];
          let cx = p[0] * this.real_width + this.offset_x;
          let cy = p[1] * this.real_height + this.offset_y;
          if (i === 0) {
            context.moveTo(cx, cy);
            start_x = cx;
            start_y = cy;
          }
          context.lineTo(cx, cy);
        }
        context.lineTo(start_x, start_y);
        context.stroke()



        for (let i = 0; i < points.length; i++) {
          context.beginPath()
          let p = points[i];
          let cx = p[0] * this.real_width + this.offset_x;
          let cy = p[1] * this.real_height + this.offset_y;

          // if (this.btn_type == 8) { //修改点，判断距离
          //   let tmp_x = this.mouseXRate * this.real_width;//+this.offset_x;
          //   let tmp_y = this.mouseYRate * this.real_height;//+this.offset_y;
          //   if (Math.abs(cx - tmp_x) < 30 && Math.abs(cy - tmp_y) < 30) {
          //     context.fillStyle = "#ff0000"; //图形填充颜色
          //   }
          //   else {
          //     context.fillStyle = color;
          //   }
          // }
          // else {
          //   context.fillStyle = color;
          // }
          context.arc(cx, cy, 7, 0, 360);
          context.fill(); //画实心圆 
          context.closePath();
        }



      }

      //绘制修改点
      if (this.btn_type == 8 && this.selectedPoint.length > 0) { //修改点，判断距离
        context.fillStyle = "#ff0000"; //图形填充颜色
        context.beginPath(); // 开始画图
        var selected_point = this.selectedPoint;
        var type = selected_point[0];
        var xx, yy = 0;
        if (selected_point.length == 2) {
          let index = selected_point[1];
          xx = this.currentLabelList[type][index][0];
          yy = this.currentLabelList[type][index][1];
        }
        else {
          let index1 = selected_point[1];
          let index2 = selected_point[2];
          xx = this.currentLabelList[type][index1]["points"][index2][0];
          yy = this.currentLabelList[type][index1]["points"][index2][1];
        }
        xx = xx * this.real_width + this.offset_x;
        yy = yy * this.real_height + this.offset_y;

        context.arc(xx, yy, 9, 0, 360);
        context.fill(); //画实心圆 
        context.closePath();
      }


    },
    zoomToLabel: function (id) {
      this.currentHighlightId = id;
      this.drawImageAndPoints();
    },
    removeLabel: function (id) {
      this.$confirm('确定要删除该数据吗？', {
        type: "warning",
        beforeClose: async (action, instance, done) => {
          if (action === "confirm") {
            try {
              if (id == 10000) {
                this.currentLabelList["points"] = [];
              }
              for (let i = 0; i < this.currentLabelList["rects"].length; i++) {
                if (this.currentLabelList["rects"][i]["id"] == id) {
                  this.currentLabelList["rects"].splice(i, 1);
                }
              }
              for (let i = 0; i < this.currentLabelList["polys"].length; i++) {
                if (this.currentLabelList["polys"][i]["id"] == id) {
                  this.currentLabelList["polys"].splice(i, 1);
                }
              }
              this.drawImageAndPoints();
              done();
            }
            catch {
              done();
            }
          }
          else {
            done();
          }
        }
      })
    },
    removeClass: function (id) {
      var self = this;
      this.$confirm('确定要删除该数据吗？', {
        type: "warning",
        beforeClose: async (action, instance, done) => {
          if (action === "confirm") {
            // 调取接口
            for (let i = 0; i < this.classList.length; i++) {
              if (this.classList[i]["id"] == id) {
                this.classList.splice(i, 1);
              }
            }
            self.drawImageAndPoints();
            done();
          }
          else {
            done();
          }
        }
      })
    },
    formatArray: function (data, num, index) {
      var tmp = []
      for (var i = 0, len = data.length; i < len; i += num) {
        tmp.push(data.slice(i, i + num))
      }
      return tmp[index]
    },
    getColorByClass(class_name) {
      for (let key in this.classList) {
        if (this.classList[key]["class_name"] == class_name) {
          return this.classList[key]["color"]
        }
      }
    },
    handleClose: function (value) {
      this.dialogVisible = false;
      var newArr = [];
      var id = 0;
      switch (this.btn_type) {
        case 3:
          if (value == 0) {
            this.current_poly_points = [];
          }
          else {
            for (let i = 0; i < this.current_poly_points.length; i++) {
              newArr[i] = [...this.current_poly_points[i]];
            }
            let class_name = this.selected_class_name;
            if (this.currentLabelList["rects"].length > 0) {
              id = this.currentLabelList["rects"][this.currentLabelList["rects"].length - 1].id + 1
            }
            id += 1000;
            this.currentLabelList["rects"].push({ "id": id, "class_name": class_name, "points": newArr });
            this.current_poly_points = [];
            this.drawImageAndPoints();
          }
          break;
        case 4:
          if (value == 0) {
            this.current_poly_points = [];
          }
          else {
            for (let i = 0; i < this.current_poly_points.length; i++) {
              newArr[i] = [...this.current_poly_points[i]];
            }
            let class_name = this.selected_class_name;
            if (this.currentLabelList["polys"].length > 0) {
              id = this.currentLabelList["polys"][this.currentLabelList["polys"].length - 1].id + 1
            }
            id += 5000;
            this.currentLabelList["polys"].push({ "id": id, "class_name": class_name, "points": newArr });
            this.current_poly_points = [];
            this.drawImageAndPoints();
          }
          break;
      }
    },
    addClass: function () {
      var max_val = 0;
      for (let i = 0; i < this.classList.length; i++) {
        if (this.classList[i].name == this.input_class.trim()) {
          this.$message({ message: '重复添加', duration: 1000 });
          return;
        }
        if (this.classList[i].id > max_val) {
          max_val = this.classList[i].id;
        }
      }
      max_val += 1;
      let item = { "id": max_val, "name": this.input_class, "color": this.color_init }
      this.classList.push(item);
      this.drawImageAndPoints();
      this.$message({
        message: '成功添加',
        type: 'success',
        duration: 1000
      });
      this.input_class = "";
    },
    color16: function () {//十六进制颜色随机
      var r = Math.floor(Math.random() * 256);
      var g = Math.floor(Math.random() * 256);
      var b = Math.floor(Math.random() * 256);
      //var color = '#'+r.toString(16)+g.toString(16)+b.toString(16);
      var color = '#' + (Array(6).join(0) + (r.toString(16) + g.toString(16) + b.toString(16))).slice(-6);
      return color;
    },
    deepCopy: function (obj) {
      return JSON.parse(JSON.stringify(obj));
    }
  },
  watch: {
    currentImgIndex: function () {
      this.init_image();
    },
    color_init: function () {
      this.drawImageAndPoints();
    }
    // currentLabelList:function(){
    //   console.log("5555555");
    //   this.allLabelList[this.indexInAllLabel]=this.deepCopy(this.currentLabelList);
    // }
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
  height: 30px !important;
  text-align: center;
  line-height: 30px;
  background-color: #409EFF;
  color: #303133;
  font-size: small;
  border-bottom: 1px solid #DCDFE6;
}

.main-container {
  display: flex;
  flex-direction: row;
  height: 100%;
  overflow: hidden;
  /* need*/
}

.markdown {
  display: flex;
  flex-direction: column;
  flex: 1;
  height: 100%;
  border-right: 1px solid #DCDFE6;
  overflow: hidden;
  color: #303133;
  font-size: small;
}

.title-left {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 10px;
  line-height: 10px;
  text-align: center !important;
  border-bottom: 1px solid #DCDFE6;
  padding: 5px;
  background-color: rgb(255, 255, 255);
  color: #303133;
  font-size: small;
}

.title-center {
  display: block;
  height: 10px;
  line-height: 10px;
  text-align: center !important;
  border-bottom: 1px solid #DCDFE6;
  padding: 5px;
  background-color: rgb(255, 255, 255);
  color: #303133;
  font-size: small;
  padding-right: 40px;
}

.title {
  display: block;
  height: 10px;
  line-height: 10px;
  text-align: center !important;
  border-bottom: 1px solid #DCDFE6;
  padding: 5px;
  background-color: rgb(255, 255, 255);
  color: #303133;
  font-size: small;
}

.fold-left {
  margin: 0 !important;
  padding: 0 !important;
  border-radius: 2px !important;
  border: 1px solid white !important;
  border: none;
  background-color: rgb(255, 255, 255);
  background-repeat: no-repeat;
  background-position: center;


  display: inline-block;
  width: 24px !important;
  height: 18px !important;
  background-image: url('../assets/fold.png');
  background-size: 80% auto;
}

.fold-left:hover {
  background-color: rgb(179, 216, 255) !important;
  border: 1px solid #E4E7ED !important;
}

.panel-zhanwei {
  width: 20px;
  height: 20px;
}

.markdown-container {
  flex: 1;
  padding-left: 10px;
  padding-right: 10px;
  background-color: rgb(255, 255, 255);
}

.VueMarkdown {
  margin: 0;
}

.image_flag {
  flex: 4;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.toolbar-canvas {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: row;
}

.center-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: rgb(40, 40, 40);
  padding: 0;
}

.canvas-container {
  position: relative;
  flex: 1;
}

.canvas1-container {
  height: 100%;
  width: 100%;
}

#canvas {
  width: 100% !important;
  height: 100% !important;
  background-color: #f0f2f5;
  /*避免canvas flex=1*/
}

.canvas2-container {
  position: absolute;
  /*父窗体relative,本窗体absolute*/
  bottom: 5px;
  left: 5px;
  z-index: 100;
  border: 1px solid #DCDFE6;
}

#canvas-small {
  width: 100%;
  height: 100%;
  background-color: red;
  border-radius: 2px;
}



.toolbar {
  display: flex;
  flex-direction: column;
  align-items: center;
  border-right: 1px solid #DCDFE6;
  background-color: rgb(255, 255, 255);
  width: 41px;
  padding-top: 10px;
}

.tool-btn {
  margin: 0 !important;
  padding: 4px !important;
  width: 30px;
  height: 24px;
  border-radius: 2px !important;
  border: 1px solid white;
  background-color: white;
  background-repeat: no-repeat;
  background-position: center;
  margin-bottom: 9px !important;
  background-size: contain;
}

.tool-btn:hover {
  background-color: rgb(179, 216, 255) !important;
  border: 1px solid #DCDFE6 !important;
}

.tool-btn-pressed {
  background-color: rgb(179, 216, 255) !important;
  border: 1px solid #DCDFE6 !important;
}


.tool-btn-hand {
  background-image: url('../assets/hand.png');
}

.tool-btn-modify {
  background-image: url('../assets/modify.png');
}

.fold-right {
  background-image: url('../assets/fold-right.png');
  background-size: 70% auto;
  padding: 3px;
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



.carousel-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  height: 100px !important;
  background-color: rgb(255, 255, 255);
  border-top: 1px solid #DCDFE6;
}

.prev {
  width: 27px;
  height: 27px;
  border-radius: 50%;
  background-color: white;
  margin-right: 7px;
  margin-left: 7px;
}

.prev:hover {
  background-color: #ecf5ff;
}

.next:hover {
  background-color: #ecf5ff;
}

.next {
  width: 27px;
  height: 27px;
  border-radius: 50%;
  margin-right: 7px;
  margin-left: 6px;
  background-color: white;
}

.imgdet {
  flex: 1;
}


.carousel-image {
  width: 100%;
  height: 100px !important;
  border: 1px solid rgb(255, 255, 255);
  box-sizing: border-box;
  -webkit-user-drag: none;
}


.carousel-image:hover {
  border: 1px solid rgb(0, 255, 0) !important;
}

.carousel-image-selected {
  border: 1px solid rgb(0, 255, 0) !important;
}

.class_label {
  flex: 1;
  height: 100%;
  border-left: 1px solid #DCDFE6;
  display: flex;
  flex-direction: column;
  color: #606266;
  font-size: small;
  background-color: rgb(255, 255, 255) !important;
}

.add-class-input {
  background-color: white;
  border: 1px solid #F2F6FC;
  color: #606266;
  margin-top: 5px;
}

.add-class-input:focus {
  outline: none;
  border: 1px solid #DCDFE6;
}



.class_label ul {
  flex: 1;
  overflow-y: scroll;
  margin-right: -17px;
  padding-inline-start: 0px !important;
  padding: 0px !important;
  margin-block-start: 0 !important;
  margin-block-end: 0 !important;
}

.class_label li {
  display: flex;
  list-style-type: none;
  height: 22px;
  vertical-align: center;
  line-height: 22px !important;
  margin-left: 1px !important;
  color: #606266;
}

.class_label li:hover {
  background-color: #eef5fe;
}

.colorbar {
  display: block;
  width: 12px;
  height: 22px;
  border: none !important;
  padding: none !important;
  margin-right: 5px;
}

.label-name {
  flex: 1;
  color: #303133;
}


.dialog-container {
  display: flex;
  flex-direction: column;
  user-select: none !important;
}


.corlor_class_container {
  display: flex !important;
  flex-direction: row !important;
  flex-wrap: wrap !important;
  align-items: flex-start !important;
  border: 1px solid #dddfe6;
  margin-top: 15px;
  max-height: 200px;
  overflow-y: scroll;
}

.color_class {
  height: 30px !important;
  flex: 0 0 20%;
  border: 1px solid #DCDFE6!important; 
  border-top: none;
  border-left: none;
  line-height: 30px !important;
  text-align: center;
  color: #606266;
  box-sizing: border-box;
}

.color_class_selected {
  border: 1px solid #409EFF!important;
}

.color_class:hover {
  border: 1px solid #409EFF!important;
}



.label-some {
  margin-left: 1px;
  display: flex;
  flex-direction: row;
  align-items: center;
}

.label-class-point {
  width: 18px;
  height: 18px;
  background-image: url('../assets/point2.png');
  background-size: cover;
}

.label-class-rect {
  width: 18px;
  height: 18px;
  background-image: url('../assets/rect2.png');
  background-size: cover;
}

.label-class-poly {
  width: 18px;
  height: 18px;
  background-image: url('../assets/poly2.png');
  background-size: cover;
}

.label-classname {
  flex: 1;
  margin-left: 3px;
}

.label-remove {
  width: 50px;
  color: #909399;
  font-size: x-small;
}

.label-remove:hover {
  color: #579ff8;
}

.grid-content {
  height: 100%;
}

.el-col {
  border-radius: 4px;
}
</style>


<style>
.el-dialog {
  user-select: none !important;
  -webkit-touch-callout: none !important;
  -webkit-user-select: none !important;
  -moz-user-select: none !important;
  -ms-user-select: none !important;
}

.el-dialog__body {
  padding: 20px 20px !important;
}

.el-select-dropdown__item span {
  user-select: none !important;
  -webkit-touch-callout: none !important;
  -webkit-user-select: none !important;
  -moz-user-select: none !important;
  -ms-user-select: none !important;
}

.el-color-picker__trigger {
  border: none !important;
  height: 20px !important;
  width: 20px !important;
  padding: 0px !important;
}

.el-color-picker--small {
  height: 20px !important;
  width: 20px !important;
  padding: 0px !important;
}

.el-color-picker__icon {
  display: none !important;
}
</style>