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
          <div class="image bg-70"></div>
          <div class="flag bg-40"></div>
        </div>
      </el-col>
      <el-col :span="4">
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
      markdown: 'Hello, **Vue Markdown**!'
    }
  },
  props: {
    msg: String
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
  color:rgb(240,240,240);
  padding-left: 10px;
  padding-right: 10px;
  font-size: small;
}

.VueMarkdown{
  margin:0;
  height:100%;
}

.image_flag {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.image {
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
