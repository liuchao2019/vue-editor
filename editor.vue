<template lang="html">
  <div class="editor">
    <div ref="toolbar" class="toolbar"></div>
    <div ref="editor" class="text"></div>
  </div>
 </template>

<script>
  import E from "wangeditor";
  // import constant from "@/api/constant";

  export default {
    name: "editor",
    data() {
      return {
        editor: null,
        info_: null,
        firstFill: false // 是否已填充取回来的数据
      };
    },

    model: {
      prop: "value",
      event: "change"
    },

    props: {
      value: {
        type: String,
        default: ""
      },

      isClear: {
        type: Boolean,
        default: false
      },

      directory: {
        // 文件上传目录
        type: String,
        default: "information"
      },
    },

    watch: {
      isClear(val) {
        // 触发清除文本域内容
        if (val) {
          this.editor.txt.clear();
          this.info_ = null;
        }
      },

      value: function(val) {
        /**
         * 实现v-model
         * 主要用于编辑时填充取回来的数据
         */
        if (val && !this.firstFill) {
          this.editor.txt.html(val);
          this.firstFill = true;
        }
      }
    },

    mounted() {
      this.seteditor();
      this.editor.txt.html(this.value);
    },

    methods: {
      seteditor() {
        let baseURL = window.__config__.open_host;
        this.editor = new E(this.$refs.toolbar, this.$refs.editor);
        this.editor.customConfig.uploadImgShowBase64 = true // base 64 存储图片
        // this.editor.customConfig.uploadImgServer = baseURL + constant.uploadImage; // 配置服务器端地址
        // this.editor.customConfig.uploadImgParams = {
        //   // fromdata
        //   // 版本 >=v3.1.1 ，属性值不会自动 encode ，如有需要自己手动 encode
        //   directory: this.directory || "information",
        // };
        this.editor.customConfig.uploadImgHeaders = {
          // 自定义 header
          Authorization: "Basic c2Vlc2hpb25fVXNlcm5hbWU6c2Vlc2hpb25fUHN3"
        };
        this.editor.customConfig.uploadFileName = "files"; // 后端接受上传文件的参数名
        this.editor.customConfig.uploadImgMaxSize = 10 * 1024 * 1024; // 将图片大小限制为 10M
        // this.editor.customConfig.uploadImgMaxLength = 6 // 限制一次最多上传 6 张图片
        this.editor.customConfig.uploadImgTimeout = 60 * 60; // 设置超时时间
        this.editor.customConfig.zIndex = 100;
        this.editor.customConfig.pasteIgnoreImg = true; // 不允许粘贴图片

        // 配置菜单
        this.editor.customConfig.menus = [
          "head", // 标题
          "bold", // 粗体
          "fontSize", // 字号
          "fontName", // 字体
          "italic", // 斜体
          "underline", // 下划线
          "strikeThrough", // 删除线
          "foreColor", // 文字颜色
          "backColor", // 背景颜色
          "link", // 插入链接
          "list", // 列表
          "justify", // 对齐方式
          "quote", // 引用
          "emoticon", // 表情
          "image", // 插入图片
          "table", // 表格
          "video", // 插入视频
          "code", // 插入代码
          "undo", // 撤销
          "redo" // 重复
        ];

        /**
         * @param {Object} xhr XMLHttpRequst 对象
         * @param {Object} editor 编辑器对象
         * @param {Object} result 服务器端返回的结果
         * @param {Function} insertImg 插入图片的函数
         */
        this.editor.customConfig.uploadImgHooks = {
          fail: (xhr, editor, result) => {
            // 插入图片失败回调
            this.$Message.error("图片插入失败");
          },
          success: (xhr, editor, result) => {
            // 图片上传成功回调
          },
          timeout: (xhr, editor) => {
            // 网络超时的回调
            this.$Message.error("图片上传网络超时");
          },
          error: (xhr, editor) => {
            // 图片上传错误的回调
            this.$Message.error("图片插入失败");
          },
          customInsert: (insertImg, result, editor) => {
            // 图片上传成功,插入图片的回调
            insertImg(result.data);
          }
        };
        this.editor.customConfig.onchange = html => {
          this.info_ = html; // 绑定当前逐渐地值
          this.$emit("change", this.info_); // 将内容同步到父组件中
        };
        // 创建富文本编辑器
        this.editor.create();
        /*初始内容*/
        this.editor.txt.html(this.value);
      },
    }
  };
</script>

<style lang="css">
.editor {
  /* width: 80%;
  margin: 0 auto; */
  max-width: 700px;
}
.toolbar {
  border: 1px solid #ccc;
}
.text {
  border: 1px solid #ccc;
  height: 450px;
}
</style>