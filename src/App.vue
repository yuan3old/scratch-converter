<template>
  <div id="app">
    <img src="./assets/logo.png" class="logo" /><br />
    <span class="text-title">Scratch Converter</span>
    在线转换 Scratch 作品文件为网页或者软件<br />

    <template v-if="showConfig == null">
      <button @click="ChooseFile">选择一个 Scratch 作品文件</button>
    </template>
    <template v-else-if="showConfig == true">
      <button @click="ConvertToHTML('html')">转换为 HTML 网页</button>
      &nbsp;
      <button @click="ConvertToHTML('exe')">转换为 EXE 软件</button>
    </template>
    <template v-else> ✅ 转换完成！ </template>
    <br />

    <template v-if="downloadProgress != null && downloadProgress < 100">
      <progress
        :value="downloadProgress"
        max="100"
        style="width: 10em"
      ></progress>
      下载资源中 . . .
    </template>
    <template v-else-if="downloadProgress != null && downloadProgress >= 100">
      <progress id="pack" max="100" style="width: 10em"></progress>
      打包中 . . .
    </template>

    <div class="convert-config" v-show="showConfig">
      <table style="display: inline-block">
        <tr>
          <td>网页标题</td>
          <td><input v-model="convertConfig.title" /></td>
        </tr>
        <tr>
          <td style="line-height: 1em">「用户名」模块的返回值</td>
          <td><input v-model="convertConfig.username" /></td>
        </tr>
        <tr>
          <td style="line-height: 1em">
            兼容模式<br />
            <small>开启则限制 30 FPS</small>
          </td>
          <td>
            <input type="checkbox" v-model="convertConfig.compatibilityMode" />
          </td>
        </tr>
        <tr>
          <td>加速模式</td>
          <td><input type="checkbox" v-model="convertConfig.turboMode" /></td>
        </tr>
        <tr>
          <td>加载进度条颜色</td>
          <td>
            <input
              type="color"
              value="#23ABF2"
              v-model="convertConfig.progressBarColour"
            />
          </td>
        </tr>
        <tr>
          <td>自动开始</td>
          <td>
            <input type="checkbox" checked v-model="convertConfig.autoStart" />
          </td>
        </tr>
        <tr>
          <td>显示全屏摁钮</td>
          <td>
            <input type="checkbox" checked v-model="convertConfig.fullscreen" />
          </td>
        </tr>
        <tr>
          <td style="line-height: 1em">
            去除限制<br />
            <small>去除列表、克隆体限制</small>
          </td>
          <td>
            <input type="checkbox" checked v-model="convertConfig.noLimits" />
          </td>
        </tr>
        <tr>
          <td style="line-height: 1em">隐藏鼠标光标</td>
          <td>
            <input type="checkbox" checked v-model="convertConfig.noCursor" />
          </td>
        </tr>
      </table>
    </div>
    <p style="color: gray;line-height: 1.5em;font-size:0.75em;">
      By Scratch Bar Open Source, Under the MIT LINENCE.<br />
      Used <a href="https://github.com/vuejs/vue">Vue</a> ( MIT LINENCE ), <a href="https://github.com/SheepTester/htmlifier">Htmlifier</a> ( MIT LINENCE ), <a href="https://github.com/Stuk/jszip">Jszip</a> ( MIT LINENCE & GPL v3 ), <a href="https://github.com/axios/axios">Axios</a> ( MIT LINENCE ) .<br/>
      Contributors : <a href="https://github.com/waterblock79">waterblock79</a>
    </p>

    <!-- 这部分不会显示 -->
    <iframe
      src="htmlifier-offline.html"
      id="htmlifier"
      style="display: none"
    ></iframe>
    <button ref="fch" id="fch" style="display: none">FinishConvertHTML</button>
    <a ref="downloadHTML"></a>
    <!---->
  </div>
</template>

<script>
const zip = require("jszip");
const axios = require("axios");

export default {
  name: "App",

  data: function () {
    return {
      convertConfig: {
        title: "My Scratch Project Meow~", // 网页标题
        username: "ScratchCat", // 用户名
        compatibilityMode: false, // 兼容模式（开启则限制 30 FPS ）
        turboMode: false, // 加速模式
        favicon: null, // 图标
        progressBarColour: "#23ABF2", //进度条颜色
        autoStart: true, // 自动开始
        fullscreen: true, // 显示全屏摁钮
        noLimits: true, // 去除克隆体、列表等的限制
        noCursor: false, // 隐藏光标
      },
      showConfig: null,
      downloadProgress: null,
      by: "scratch-bar",
    };
  },

  methods: {
    ChooseFile: function () {
      let frame = window.frames["htmlifier"].contentWindow;
      //选择一个文件
      frame.document.getElementById("file").click();
      //当选择完成
      frame.document.getElementById("file").onchange = () => {
        this.showConfig = true;
      };
    },

    ConvertToHTML: function (type) {
      let frame = window.frames["htmlifier"].contentWindow;
      //传入转换设置
      //console.log(this.convertConfig);
      frame.window.convertConfig = this.convertConfig;
      //开始转换
      frame.document.getElementById("htmlify").click();
      this.$refs["fch"].addEventListener("click", () => {
        this.FinishConvertHTML(type);
      });
    },

    FinishConvertHTML: function (type) {
      if (type == "html") {
        let p = this.$refs["downloadHTML"];
        p.href = URL.createObjectURL(window.htmlBlob);
        p.download = `${this.convertConfig.title}.html`;
        p.click();
        this.showConfig = false;
      }
      if (type == "exe") {
        axios
          .get("./nwjs-v0.54.1-win-x64.zip", {
            onDownloadProgress: (progress) => {
              this.downloadProgress = Math.floor(
                (progress.loaded / progress.total) * 100
              );
            },
            responseType: "blob",
          })
          .then(function (response) {
            let final_zip = new zip();
            // more files !
            final_zip.loadAsync(response.data).then(function (zip) {
              // you now have every files contained in the loaded zip
              zip
                .folder("nwjs-v0.54.1-win-x64")
                .file("project.html", window.htmlBlob);
              zip.folder("nwjs-v0.54.1-win-x64").file(
                "package.json",
                `{
                    "name": "Scratch Project",
                    "main": "project.html"
                  }
                `,
                { binary: false }
              );
              window.packProgress = 0;
              zip
                .generateAsync(
                  { type: "blob" },
                  function updateCallback(metadata) {
                    document.getElementById("pack").value = metadata.percent;
                    //如果使用 this，会报 undefind，百度过了，并没有找到有效的解决方案
                  }
                )
                .then(function (blob) {
                  let p = document.createElement("a");
                  p.href = URL.createObjectURL(blob);
                  p.download = `Project.zip`;
                  p.click();
                  this.showConfig = false;
                });
            });
          });
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
  font-weight: 50;
  line-height: 3em;
}

.logo {
  height: 7.5em;
  width: 7.5m;
  margin-bottom: 1em;
}

.text-title {
  display: block;
  font-size: xx-large;
}

.convert-config {
  margin-top: 1em;
  line-height: 2.5em;
}

button {
  padding: 10px 14px 10px 14px;
  background-color: #e5e5e5;
  color: black;
  border: none;
  border-radius: 3px;
  transition: background-color 0.5s;
}

button:hover {
  background-color: #d5d5d5;
}

button:active {
  background-color: #cfcfcf;
}

input {
  border-top: none;
  border-left: none;
  border-right: none;
  border-bottom: soild #2c3e50;
  outline: none;
  margin-left: 1em;
  font-size: 0.9em;
}

input:focus {
  border-bottom-color: #263646;
}

progress {
  margin-right: 1em;
}

a{
  text-decoration: none;
  color: rgb(36,64,179)
}
</style>
