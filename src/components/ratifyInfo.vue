<template>
    <div style="margin:auto">
        <div v-if="ratifyData">
            <h2 style="margin: 10px">{{ ratifyData['ratifyNo'] }} {{ ratifyData['projectName'] }}</h2>
            <Collapse v-model="panel" simple style="text-align: left">
                <Panel name="baseInfo">
                    <b>基本信息</b>
                    <div slot="content">
                        <Row>
                            <Col span="4">批准号</Col>
                            <Col span="20">{{ ratifyData['ratifyNo'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">项目名称</Col>
                            <Col span="20">{{ ratifyData['projectName'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">项目类别</Col>
                            <Col span="20">{{ type_map[ratifyData['projectType']] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">申请代码</Col>
                            <Col span="20">{{ ratifyData['code'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">项目负责人</Col>
                            <Col span="20"><a :href="`http://output.nsfc.gov.cn/projectAdmin/${ratifyData['projectAdminID']}`"
                                              target="_blank" rel="noreferrer">{{ ratifyData['projectAdmin'] }}</a></Col>
                        </Row>
                        <Row>
                            <Col span="4">负责人职称</Col>
                            <Col span="20">{{ ratifyData['adminPosition'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">研究期限</Col>
                            <Col span="20">{{ ratifyData['researchTimeScope'].replace(/ 00:00:00.0/ig,'').replace('到', ' 至 ') }}
                            </Col>
                        </Row>
                        <Row>
                            <Col span="4">支持经费</Col>
                            <Col span="20">{{ ratifyData['supportNum'] }}（万元）</Col>
                        </Row>
                    </div>
                </Panel>
                <Panel name="abstractInfo">
                    <b>项目摘要</b>
                    <div slot="content">
                        <Row>
                            <Col span="4">中文摘要</Col>
                            <Col span="20">{{ ratifyData['projectAbstractC'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">英文摘要</Col>
                            <Col span="20">{{ ratifyData['projectAbstractE'] }}</Col>
                        </Row>
                        <Row>
                            <Col span="4">结题摘要</Col>
                            <Col span="20">{{ ratifyData['conclusionAbstract'] }}</Col>
                        </Row>
                    </div>
                </Panel>
                <Panel name="resultsInfo">
                    <b>成果</b>
                    <div slot="content">
                        <Table :columns="result_table_columns" :data="result_table_data"/>
                    </div>
                </Panel>
            </Collapse>
        </div>
        <div v-else-if="ratifyDataError">
            项目可能不存在，请重新检查网页 <a :href="`http://output.nsfc.gov.cn/conclusionProject/${ratify}`" target="_blank" rel="noreferrer">http://output.nsfc.gov.cn/conclusionProject/{{ratify}}</a>
        </div>
        <div v-else>
            正在请求项目信息，请稍等..........
        </div>
    </div>
</template>

<script>
  const axios = require('axios').default;
  const no_cors_api_point = 'https://cors.rhilip.info/';

  export default {
    name: "ratifyInfo",
    props: {
      ratify: String
    },
    data() {
      return {
        ratifyData: null,
        ratifyDataError : null,
        'panel': ['baseInfo', 'abstractInfo'],
        'type_map': {
          "630": "青年科学基金项目",
          "631": "地区科学基金项目",
          "218": "面上项目",
          "632": "海外及港澳学者合作研究基金",
          "220": "重点项目",
          "222": "重大项目",
          "339": "重大研究计划",
          "429": "国家杰出青年科学基金",
          "432": "创新研究群体项目",
          "433": "国际(地区)合作与交流项目",
          "649": "专项基金项目",
          "579": "联合基金项目",
          "70": "应急管理项目",
          "7161": "科学中心项目",
          "635": "国家基础科学人才培养基金",
          "2699": "优秀青年科学基金项目",
          "8531": "专项项目",
          "51": "国家重大科研仪器设备研制专项",
          "52": "国家重大科研仪器研制项目",
          "2733": "海外或港、澳青年学者合作研究基金"
        },
        'result_table_columns': [
          {title: "序号", key: "id", sortable: true, sortType: 'asc', width: 85},
          {
            title: "标题", key: "title",
            render: (h, params) => {
              return h('a', {
                attrs: {
                  href: `http://output.nsfc.gov.cn/resultInfo/${params.row._id}`,
                  target: '_blank',
                  rel: "noreferrer"
                }
              }, params.row.title);
            }
          },
          {title: "类型", key: "type", sortable: true, width: 100},
          {title: "作者", key: "author", sortable: true, ellipsis: true}
        ],
        'result_table_data': []
      }
    },
    methods: {
      cleanLastRatifyInfo () {
        this.$emit('update_ratify_data', null);
        this.$emit('update_ratify_img_list', []);
      },

      getRatifyInfo () {
        let vm = this;

        this.$Loading.start();
        this.$emit('disable_next',true);
        axios.get(no_cors_api_point,{
          params: {
            apiurl : `http://output.nsfc.gov.cn/baseQuery/data/conclusionProjectInfo/${this.ratify}`
          }
        }).then(function (response) {
          vm.$Loading.finish();
          if (response.status === 200) {
            let data = response.data;
            if (data.code === 200) {
              vm.ratifyData = data.data;
              vm.$emit('update_ratify_data', vm.ratifyData);
              vm.getResultsList();
            }
          }

          vm.$emit('disable_next', !vm.ratifyData);
        }).catch(function (error) {
          console.log(error);
          vm.ratifyDataError = error;
          vm.$Loading.error();
        });
      },

      getResultsList() {
        let l = [];
        for (let i = 0; i < this.ratifyData['resultsList'].length; i++) {
          let conv = this.ratifyData['resultsList'][i]['result'];
          l.push({'id': parseInt(conv[0]), '_id': conv[1], 'title': conv[2], 'type': conv[3], 'author': conv[4]})
        }
        this.result_table_data = l;
      }
    },
    mounted() {
      this.cleanLastRatifyInfo();
      this.getRatifyInfo();
    }
  }
</script>