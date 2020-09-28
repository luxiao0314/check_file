<template>
  <div id="app">
    <br />
    <el-input
      v-model="input"
      placeholder="请输入机型"
      style="width: 150px; margin-right: 20px"
    />
    <el-radio v-model="radio" label="正常">正常</el-radio>
    <el-radio v-model="radio" label="维修">维修</el-radio>
    <br /><br />

    <div>
      <span>上传文件: </span>
      <input type="file" ref="refFile1" @change="fileUpload" />
      <button @click="exportData">导出</button>
    </div>

    <br /><br />
    <div v-for="(line, index) in checkResultList" :key="index">{{ line }}</div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      standardList: [],
      checkResultList: [],
      filename: "",
      radio: "正常",
      input: "",
    };
  },
  methods: {
    //标准文件上传
    fileUpload() {
      this.standardList = [];
      let files = this.$refs.refFile1.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);

        this.filename = file.name;
        this.filter(file, (items) => {
          //标准数据不包含就添加
          items.forEach((it) => {
            if (!this.standardList.includes(it)) {
              this.standardList.push(it);
            }
          });
        });
      }

      console.log("标准数据:");
      console.log(this.standardList);
    },

    //导出数据
    exportData() {
      let results = new Map();

      //获取第一个=号,转map去重
      this.standardList.forEach((it) => {
        let index = it.indexOf("=");
        let key = it.substring(0, index);
        let value = it.substring(index + 1, it.length);
        results.set(key, value);
      });

      console.log(results);

      this.checkResultList = [];
      this.checkResultList.push("IMEI,");
      this.checkResultList.push("机型,");
      this.checkResultList.push("正常与否,");
      for (let [key, value] of results) {
        this.checkResultList.push(key + ",");
      }

      this.checkResultList.push("\r\n");

      this.checkResultList.push(this.filename.replace(".txt", "") + ",");
      this.checkResultList.push(this.input + ",");
      this.checkResultList.push(this.radio + ",");
      for (let [key, value] of results) {
        this.checkResultList.push(value.replace(/,/g, '","') + ","); //  /,/g:全局替换逗号
      }

      //定义文件内容，类型必须为Blob 否则createObjectURL会报错
      let content = new Blob(this.checkResultList, {
        endings: "transparent",
      });

      //生成url对象
      let urlObject = window.URL || window.webkitURL || window;
      let url = urlObject.createObjectURL(content);
      //生成<a></a>DOM元素
      let el = document.createElement("a");
      //链接赋值
      el.href = url;
      el.download = this.filename.replace(".txt", ".csv");
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
        let resultList = [];

        //每行过滤
        let lineFilter = (it) =>
          it != "" &&
          it.includes("=") &&
          !it.includes("Activity") &&
          !it.includes("Time") &&
          !it.includes(">") &&
          !it.includes("<");

        //获取每行数据并过滤
        let line = e.target.result.split("\r\n").filter((it) => lineFilter(it));

        line.forEach((element) => {
          //字段过滤
          let filter = (it) =>
            it != "" &&
            it.includes("=") &&
            !it.includes("taskId") &&
            !it.includes("Task") &&
            !it.includes(":") &&
            !it.includes("mStackId") &&
            !it.includes("pid") &&
            !it.includes("mLayoutSeq") &&
            !it.includes("Window") &&
            !it.includes("/");

          if (element.includes("Rect") || element.includes("DisplayInfo")) {
            resultList.push(
              element
                .replace("    ", "")
                .replace("      ", "")
                .replace("    ", "")
                .replace("  ", "")
            );
          } else {
            let elementlist = element.split(" ").filter((it) => filter(it));
            resultList.splice(resultList.length, 0, ...elementlist);
          }
        });

        callback(new Set(resultList));
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
