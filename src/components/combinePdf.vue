<template>
    <div style="margin:auto">
        <h2 style="margin: 10px">{{ info['ratifyNo'] }} {{ info['projectName'] }}</h2>
        <Button @click="convertPdf">下载</Button>
    </div>
</template>

<script>
  const axios = require('axios').default;
  const jsPDF = require('jspdf');
  const no_cors_api_point = 'https://cors.rhilip.info/';

  export default {
    name: "combinePdf",
    props: {
      info: Object,
      imgs: Array
    },
    methods: {
      convertPdf() {
        let vm = this;
        let promises = [];

        for (let i = 0; i < this.imgs.length; i++) {
          let image_data = this.imgs[i];

          if (!image_data.hasOwnProperty('blob')) {
            promises.push(
              axios.get(no_cors_api_point, {
                params: {
                  apiurl: image_data['src']
                },
                responseType: 'arraybuffer'
              }).then(response => {
                image_data['blob'] = Buffer.from(response.data, 'binary');
                vm.imgs[i] = image_data;
              })
            );
          }
        }

        Promise.all(promises).then(() => {
          let pdf = new jsPDF();
          pdf.deletePage(1);
          for (let i = 0; i < this.imgs.length; i++) {
            let image_data = this.imgs[i];
            console.log(image_data);
            pdf.addPage([image_data['width'], image_data['height']]);
            pdf.addImage(image_data['blob'], 'PNG', 0, 0);
          }
          pdf.save('test.pdf');
        });
      }
    }
  }
</script>

<style scoped>

</style>