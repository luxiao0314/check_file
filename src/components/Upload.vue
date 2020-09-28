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
      <input
        type="file"
        ref="refFile1"
        @change="fileUpload"
        multiple="multiple"
      />
      <button @click="exportAllData">导出</button>
    </div>
    <br />

    <div v-for="(line, index) in content" :key="index">{{ line }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      checkResultList: [],
      radio: "正常",
      input: "",
      content: new Map(),
      maxTitle: [],
    };
  },
  methods: {
    //标准文件上传
    fileUpload() {
      this.content = new Map();
      let files = this.$refs.refFile1.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        let filename = file.name.replace(".txt", "");

        this.filter(file, (items) => {
          let results = new Map();

          //获取第一个=号,转map去重
          items.forEach((it) => {
            let index = it.indexOf("=");
            let key = it.substring(0, index);
            let value = it.substring(index + 1, it.length);
            results.set(key, value);
          });

          // console.log(results);

          this.content.set(filename, results);

          //获取最长的数据
          let size = 0;
          for (let [key, value] of this.content) {
            if (size <= value.size) {
              size = value.size;
              this.maxTitle = value;
            }
          }
        });
      }

      console.log("map数据:");
      console.log(this.content);
    },

    //导出所有
    exportAllData() {
      this.checkResultList = [];
      this.checkResultList.push("IMEI,");
      this.checkResultList.push("机型,");
      this.checkResultList.push("正常与否,");
      for (let [key, value] of this.maxTitle) {
        this.checkResultList.push(key + ",");
      }

      for (let [key, value] of this.content) {
        this.checkResultList.push("\r\n");
        this.checkResultList.push(key + ",");
        this.checkResultList.push(this.input + ",");
        this.checkResultList.push(this.radio + ",");

        for (let [key1, value2] of this.maxTitle) {
          let content = value.get(key1);
          if (content == undefined) {
            this.checkResultList.push(" ,");
          } else {
            //替换引号和逗号
            content = content.replace(/\"/g, "").replace(/,/g, '","') + ","; //  /,/g:全局替换逗号
            this.checkResultList.push(content);
          }
        }
      }

      console.log(this.checkResultList);

      //定义文件内容，类型必须为Blob 否则createObjectURL会报错
      let content = new Blob(this.checkResultList);

      //生成url对象
      let urlObject = window.URL || window.webkitURL || window;
      let url = urlObject.createObjectURL(content);
      //生成<a></a>DOM元素
      let el = document.createElement("a");
      //链接赋值
      el.href = url;
      el.download = this.input + "-" + this.radio + ".csv";
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
