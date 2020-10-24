<template>
  <div id="app">
    <br />
    <el-input
      v-model="input"
      placeholder="请输入机型"
      style="width: 150px; margin-right: 20px"
    />

    <button @click="exportAllData">导出</button>
    <br /><br />

    <div>
      <span>上传正常文件: </span>
      <input
        type="file"
        ref="refFile1"
        @change="normalFileUpload"
        multiple="multiple"
      />
    </div>
    <br />

    <div>
      <span>上传维修文件: </span>
      <input
        type="file"
        ref="refFile2"
        @change="erroFileUpload"
        multiple="multiple"
      />
    </div>

    <br />
  </div>
</template>

<script>
export default {
  data() {
    return {
      checkResultList: [],
      input: "",
      normalData: new Map(),
      errorData: new Map(),
      maxData: [],
    };
  },
  methods: {
    //标准文件上传
    normalFileUpload() {
      this.normalData = new Map();
      let files = this.$refs.refFile1.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        let filename = file.name.replace(".txt", "");

        this.filter(file, (items) => {
          this.normalData.set(filename, items);
        });
      }

      console.log("标准数据:");
      console.log(this.normalData);
    },

    //异常文件上传
    erroFileUpload() {
      this.errorData = new Map();
      let files = this.$refs.refFile2.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        let filename = file.name.replace(".txt", "");

        this.filter(file, (items) => {
          //items需要为map结构
          this.errorData.set(filename, items);
        });
      }

      console.log("异常数据:");
      console.log(this.errorData);
    },

    //导出所有
    exportAllData() {
      //获取最长的数据
      let size = 0;
      for (let [key, value] of this.normalData) {
        if (size <= value.size) {
          size = value.size;
          this.maxData = value;
        }
      }

      console.log("maxdata " + this.maxData.size);

      let errorMaxdata = [];
      for (let [key, value] of this.errorData) {
        if (size <= value.size) {
          size = value.size;
          this.errorMaxdata = value;
        }
      }

      console.log("errorMaxdata " + this.maxData.size);

      if (errorMaxdata.size > this.maxData) {
        this.maxData = errorMaxdata;
      }

      this.checkResultList = [];
      this.checkResultList.push("IMEI,");
      this.checkResultList.push("机型,");
      this.checkResultList.push("正常与否,");
      for (let [key, value] of this.maxData) {
        this.checkResultList.push(key + ",");
      }

      for (let [key, value] of this.normalData) {
        this.checkResultList.push("\r\n");
        this.checkResultList.push("'" + key + ",");
        this.checkResultList.push(this.input + ",");
        this.checkResultList.push("正常,");

        for (let [key1, value2] of this.maxData) {
          let content = value.get(key1);
          if (content == undefined) {
            this.checkResultList.push(" ,");
          } else {
            //替换引号和逗号
            content = content.replace(/\"/g, "").replace(/,/g, "，") + ",";
            this.checkResultList.push(content);
          }
        }
      }

      for (let [key, value] of this.errorData) {
        this.checkResultList.push("\r\n");
        this.checkResultList.push("'" + key + ",");
        this.checkResultList.push(this.input + ",");
        this.checkResultList.push("维修,");

        for (let [key1, value2] of this.maxData) {
          let content = value.get(key1);
          if (content == undefined) {
            this.checkResultList.push(" ,");
          } else {
            //替换引号和逗号
            // content = content.replace(/\"/g, "").replace(/,/g, '","') + ","; //  /,/g:全局替换逗号
            content = content.replace(/\"/g, "").replace(/,/g, "，") + ",";
            this.checkResultList.push(content);
          }
        }
      }

      console.log(this.checkResultList);

      //定义文件内容，类型必须为Blob 否则createObjectURL会报错
      let blob = new Blob(this.checkResultList);

      //生成url对象
      let urlObject = window.URL || window.webkitURL || window;
      let url = urlObject.createObjectURL(blob);
      //生成<a></a>DOM元素
      let el = document.createElement("a");
      //链接赋值
      el.href = url;
      el.download = this.input + ".csv";
      //必须点击否则不会下载
      el.click();
      //移除链接释放资源
      urlObject.revokeObjectURL(url);
    },

    //过滤文件内容
    filter(selectedFile, callback) {
      let list = [];

      if (!selectedFile) return;

      var reader = new FileReader();

      reader.readAsText(selectedFile);

      reader.onloadend = (e) => {
        let resultList = new Map()

        //每行过滤
        let lineFilter = (it) =>it != "" && it.includes(":") &&!it.includes("running") &&!it.includes("stopped")

        //获取每行数据并过滤
        let line = e.target.result.split("\n").filter((it) => lineFilter(it));

        line.forEach((element) => {

          let elementlist = element.split(": ")

          resultList.set(elementlist[0],elementlist[1])
        });

        callback(resultList);
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
