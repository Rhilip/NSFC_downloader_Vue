<template>
    <div style="margin:auto">
        <h2 style="margin: 10px">{{ info['ratifyNo'] }} {{ info['projectName'] }}</h2>

        <div v-if="image_loading">
            <p>已经加载 {{ current }} 页......</p>
        </div>
        <div v-else-if="current > 0">
            <p>加载完成，共有 {{ current }} 页。</p>
        </div>
        <div v-else>
            未获取到任何结题报告信息，请确认该项目存在结题报告。
        </div>

        <div v-if="current !== 0">
            <Row>
                <div v-for="(img,index) in imgs" :key="index">
                    <Col span="4">
                        <img :alt="index" :src="img" @load="loadNextImage" @error="abortImageLoad" style="max-width: 100%" ref="load_img">
                    </Col>
                </div>
            </Row>
        </div>
    </div>
</template>

<script>
  export default {
    name: "conclusionImages",
    props: {
      info: Object
    },
    data () {
      return {
        imgs: [],
        current : 1,
        ratify : '',
        ratify_prefix : '',
        image_loading: true,
      }
    },
    methods: {
      loadNextImage () {
        this.$emit('disable_next', true);
        this.imgs.push(`http://output.nsfc.gov.cn/report/${this.ratify_prefix}/${this.ratify}_${this.current}.png`);
        this.current++;
      },

      abortImageLoad () {
        this.image_loading = false;

        this.imgs.pop();

        this.current = this.imgs.length;

        let imgs = [];
        for (let i = 0; i < this.$refs.load_img.length ; i++) {
          let img = this.$refs.load_img[i];
          let o = {};
          o['src'] = img.src;
          o['width'] = img.naturalWidth;
          o['height'] = img.naturalHeight;
          if (img.naturalWidth !== 0 && img.naturalHeight !== 0) {
            imgs.push(o);
          }
        }

        this.$emit('disable_next', false);
        this.$emit('update_ratify_img_list', imgs);
      },
    },
    mounted() {
      this.ratify = this.info['ratifyNo'];
      this.ratify_prefix = this.ratify.substr(0,2);
      this.loadNextImage();
    }
  }
</script>

<style scoped>

</style>