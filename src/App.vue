<template>
  <div id="app">
    <img alt="Logo" src="./assets/logo.jpg" style="width: 200px;" />
    <br>
    <br>
    <h1>国自然 结题报告 在线下载工具</h1>
    <p>
      <br>
      从 <a href="http://output.nsfc.gov.cn/" target="_blank" rel="noreferrer">科学基金共享服务网（科技成果信息系统）</a> 下载项目 PDF格式 结题报告。
      <br>
      <small>A online tool to help you download PDF format conclusion from <a href="http://output.nsfc.gov.cn/" target="_blank" rel="noreferrer">NSFC</a></small>
      <br>
    </p>
    <br>
    <Row type="flex">
      <Col span="16" push="4">
        <Divider />
        <Steps :current="current" style="  text-align: left;">
          <Step title="步骤1" content="输入项目编号" />
          <Step title="步骤2" content="检查项目信息" />
          <Step title="步骤3" content="加载项目结题报告" />
          <Step title="步骤4" content="下载PDF文件" />
        </Steps>
        <Divider />
        <Card>
          <div v-if="current === 0">
            <p>
              请输入国自然项目编号，例如项目
              <a href="http://output.nsfc.gov.cn/conclusionProject/31270544" target="_blank" rel="noreferrer">生物炭强化石油烃污染土壤生态修复及机理研究</a>
              的编号为 <code>31270544</code>
            </p>
            <br>
            <Input v-model="ratify" style="width: 500px;margin:auto" placeholder="31270544" clearable>
              <span slot="prepend">http://output.nsfc.gov.cn/conclusionProject/</span>
            </Input>
          </div>
          <div v-else-if="current === 1">
            <ratifyInfo :ratify="ratify" @update_ratify_data="updateRatifyData" @disable_next="updateLoadStatus"/>
          </div>
          <div v-else-if="current === 2">
            <conclusionImages :info="ratifyData" @update_ratify_img_list="updateRatifyImageList" @disable_next="updateLoadStatus"/>
          </div>
          <div v-else-if="current === 3">
            <combinePdf :info="ratifyData" :imgs="ratifyImageList" />
          </div>
        </Card>
        <Divider />
        <Button v-if="current !== 0" type="primary" @click="prev">上一步</Button>&nbsp;&nbsp;&nbsp;
        <Button v-if="current !== 3" type="primary" :disabled="disable_next" @click="next">下一步</Button>
      </Col>
    </Row>
    <BackTop />
  </div>
</template>

<script>
  import ratifyInfo from "@/components/ratifyInfo.vue";
  import conclusionImages from "@/components/conclusionImages.vue";
  import combinePdf from "@/components/combinePdf.vue";

export default {
  name: 'app',
  data () {
    return {
      'current' : 0,
      'ratify' : '31270544',
      'ratifyData' : null,
      'ratifyImageList' : null,
      'disable_next' : false
    }
  },
  components: {
    ratifyInfo,
    conclusionImages,
    combinePdf
  },
  methods: {
    updateRatifyData(ratifyData) {
      this.ratifyData = ratifyData
    },

    updateRatifyImageList(ratifyImageList) {
      this.ratifyImageList = ratifyImageList
    },

    updateLoadStatus(status) {
      this.disable_next = status;
    },

    prev () {
      this.current -= 1;
      if (this.current <= 0) {
        this.current = 0;
      }
      this.disable_next = false;
    },
    next () {
      if (this.current < 3) {
        this.current += 1;
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}
</style>
