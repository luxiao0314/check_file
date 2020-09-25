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
      <button @click="check">维修正常文件校验</button>
    </div>

    <br />

    <div>
      <span>上传结果文件: </span>
      <input
        type="file"
        ref="refFile3"
        @change="resultUpload"
        multiple="multiple"
      />

      <button @click="resultCheck">结果校验</button>
    </div>

    <br />

    <br /><br />

    <div v-for="(line, index) in checkList" :key="index">{{ line }}</div>

    <div v-for="(line, index) in checkResultList" :key="index">{{ line }}</div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      standardList: [],
      checkList: [],
      errlist: [],
      resultList: [],
      checkResultList: [],
    };
  },
  methods: {
    //维修文件上传
    mutileUpload() {
      this.errlist = [];
      let files = this.$refs.refFile2.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        this.filter(file, (items) => {
          //维修数据不包含就添加
          items.forEach((it) => {
            if (!this.errlist.includes(it)) {
              this.errlist.push(it);
            }
          });
        });
      }

      console.log("维修数据:");
      console.log(this.errlist);
    },

    //标准文件上传
    fileUpload() {
      this.standardList = [];
      let files = this.$refs.refFile1.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
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

    //检测
    check() {
      let minus = this.errlist.filter((x) => !this.standardList.includes(x));

      let results = new Map();

      //获取第一个=号,转map去重
      minus.forEach((it) => {
        let index = it.indexOf("=");
        let key = it.substring(0, index);
        let value = it.substring(index + 1, it.length);
        results.set(key, value);
      });

      this.checkList = [];

      //采用map对key去重
      // for (let item of results) {
      //   this.checkList.push(item);
      // }

      for (let [key, value] of results) {
        this.checkList.push(key + "=" + value);
      }

      console.log("检测结果:");
      console.log(this.checkList.sort());
    },

    //结果筛选
    resultCheck() {
      let results = new Map();

      //获取第一个=号,转map去重
      this.resultList.forEach((it) => {
        let index = it.indexOf("=");
        let key = it.substring(0, index);
        let value = it.substring(index + 1, it.length);
        results.set(key, value);
      });

      console.log(results);

      this.checkResultList = [];
      for (let [key, value] of results) {
        this.checkResultList.push(key + "=" + value);
      }
    },

    //结果文件上传
    resultUpload() {
      this.resultList = [];
      let files = this.$refs.refFile3.files;

      for (var i = 0; i < files.length; i++) {
        let file = files.item(i);
        this.resultFilter(file, (items) => {
          //维修数据不包含就添加
          items.forEach((it) => {
            if (!this.resultList.includes(it)) {
              this.resultList.push(it);
            }
          });
        });
      }

      console.log("结果筛选:");
      console.log(this.resultList.sort());
    },

    resultFilter(selectedFile, callback) {
      let list = [];

      if (!selectedFile) return;

      var reader = new FileReader();

      reader.readAsText(selectedFile);

      reader.onloadend = (e) => {
        let line = e.target.result.split("\n");

        callback(new Set(line));
      };
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
