<template>
  <div id="app">
    <br />

    <div>
      <span>上传标准文件: </span>
      <input
        type="file"
        ref="refFile1"
        @change="fileUpload"
        multiple="multiple"
      />
    </div>

    <br />

    <div>
      <span>上传维修文件: </span>
      <input
        type="file"
        ref="refFile2"
        @change="mutileUpload"
        multiple="multiple"
      />
    </div>

    <br />

    <button @click="check">校验</button>

    <br /><br />

    <div v-for="(line, index) in checkList" :key="index">{{ line }}</div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      standardList: [],
      checkList: [],
      list: [],
    };
  },
  methods: {
    //维修文件上传
    mutileUpload() {
      this.list = []
      let files = this.$refs.refFile2.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        this.upload(file, (items) => {
          this.list.splice(this.list.length, 0, ...items);
        });
      }

      console.log("维修数据:");
      console.log(this.list);
    },
    //标准文件上传
    fileUpload() {
      this.standardList = []
      let files = this.$refs.refFile1.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        this.upload(file, (items) => {
          this.standardList.splice(this.standardList.length, 0, ...items);
        });
      }

      console.log("标准数据:");
      console.log(this.standardList);
    },
    //检测
    check() {
      this.checkList = []
      this.list.forEach((it) => {
        if (!this.standardList.includes(it)) {
          this.checkList.push(it);
        }
      });

      //去重
      this.checkList = [...new Set(this.checkList)];

      //排序
      // this.checkList.sort();

      console.log("检测结果:");
      console.log(this.checkList);
    },
    upload(selectedFile, callback) {
      let list = [];

      if (!selectedFile) return;

      var reader = new FileReader();

      reader.readAsText(selectedFile);

      reader.onloadend = (e) => {
        let lineFilter = (it) =>
          it != "" && it.includes("=") && !it.includes(":");

        let line = e.target.result.split("\r\n").filter((it) => lineFilter(it));

        line.forEach((element) => {
          let filter = (it) =>
            it != "" &&
            it.includes("=") &&
            !it.includes("{") &&
            !it.includes("}") &&
            !it.includes("taskId") &&
            !it.includes("mStackId") &&
            !it.includes("pid") &&
            !it.includes("/");

          let elementlist = element.split(" ").filter((it) => filter(it));

          callback(elementlist);
        });
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
