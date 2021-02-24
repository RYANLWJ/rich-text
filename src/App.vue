<template>
  <div id="app">
    <!-- <button @click="deleteImage">Delete</button> -->
    <h1 class="header">RICH TEXT EDITOR</h1>
    <VueEditor
      class="vueEditor"
      ref="vEditor"
      v-model="content"
      use-custom-image-handler
      use-markdown-shortcuts
      @focus="onEditorFocus"
      @blur="onEditorBlur"
      @imageAdded="handleImageAdded"
      @image-removed="handleImageRemoved"
      :editorOptions="editorOption"
    />

    <div class="preview-wrap">
      <div class="title">Preview</div>
      <div class="preview"></div>
    </div>
    <div id="btn" class="button"></div>
  </div>
</template>

<script>
// import axiosInstance from "@/helpers/axios";
import "highlight.js/styles/tomorrow-night-blue.css"; // 高亮代码css
import hljs from "highlight.js"; // 高亮代码方法
import axios from "axios";
import Quill from "quill";
const AlignStyle = Quill.import("attributors/style/align");
Quill.register(AlignStyle, true);

const BlockEmbed = Quill.import("blots/block/embed");

const CLIENT_ID = "993793b1d8d3e2e";

/**
 * Customize image so we can add an `id` attribute
 */
class ImageBlot extends BlockEmbed {
  static create(value) {
    const node = super.create();
    node.setAttribute("src", value.url);
    node.setAttribute("id", value.id);
    return node;
  }

  static value(node) {
    return {
      url: node.getAttribute("src"),
      id: node.getAttribute("id"),
    };
  }
}

ImageBlot.blotName = "image";
ImageBlot.tagName = "img";
Quill.register(ImageBlot);

export default {
  data() {
    return {
      // content: "<h1>Demo</h1><h2>Take a try here ... 🧟</h2>",
      content: `<h1>Demo</h1><h2>Take a try here ... 🧟</h2><pre class="ql-syntax" spellcheck="false">const foo = <span class="hljs-keyword">function</span>(<span class="hljs-type">name</span>){
  <span class="hljs-keyword">return</span> <span class="hljs-type">name</span>
};

foo("hello world");  // <span class="hljs-keyword">function</span> "foo" <span class="hljs-keyword">called</span>

console.log(foo(hello world)) // print "hello world"

</pre>`,
      editorOption: {
        modules: {
          toolbar: [
            ["bold", "italic", "underline", "strike"],
            ["blockquote", "code-block"],
            [{ header: 1 }, { header: 2 }],
            [{ list: "ordered" }, { list: "bullet" }],
            [{ script: "sub" }, { script: "super" }],
            [{ indent: "-1" }, { indent: "+1" }],
            [{ direction: "rtl" }],
            [{ size: ["small", false, "large", "huge"] }],
            [{ header: [1, 2, 3, 4, 5, 6, false] }],
            [{ color: [] }, { background: ["blue"] }],
            [{ font: [] }],
            [{ align: [] }],
            ["clean"],
            ["link", "image", "video"],
          ],
          syntax: {
            highlight: (text) => {
              // console.log("text",text);
              return hljs.highlightAuto(text).value; // 这里就是代码高亮需要配置的地方
            },
          },
        },
      },
    };
  },
  watch: {
    content: {
      immediate: true,
      handler(v) {
        this.$nextTick(() => {
          let preview = document.querySelector(".preview");
          preview.innerHTML = v;
        });
      },
    },
  },
  create() {},
  mounted() {
    const btn = document.querySelector(".button");
    const target = document.querySelector("#app");
    let editor = document.querySelector(".vueEditor");
    let preview = document.querySelector(".preview-wrap");

    // const iw = btn.offsetLeft;
    // console.log(iw);
    // console.log(btn);
    // let X;
    // btn.addEventListener("mousedown", (e) => {
    //   X = e.clientX;
    //   btn.addEventListener("mousemove", (e) => {
    //     let gapX = e.clientX - X;

    //     console.log(iw + gapX);
    //     let left = iw + gapX;
    //     btn.style.left = left + "px";

    //     btn.addEventListener("mouseup", (e) => {
    //       btn.onmousemove = null; //把鼠标移动清除
    //       btn.onmouseup = null; //把鼠标松开清除
    //     });
    //   });
    // });
    this.drag(btn, btn, editor, preview, (res) => console.log(res));
  },
  methods: {
    handleTextChange(obj) {
      console.log("TCL: handleTextChange -> obj", obj);
    },

    onEditorBlur(quill) {
      console.log("editor blur!", quill);
    },

    onEditorFocus(quill) {
      console.log("editor focus!", quill);
    },

    deleteImage(id) {
      const ACCESS_TOKEN = "1d3ee7fe34576a55a24ec551061b2365b6ebaf23";

      axios({
        url: `https://api.imgur.com/3/image/${id}`,
        method: "DELETE",
        headers: { Authorization: "Bearer " + ACCESS_TOKEN },
      }).then((result) => console.log("DELETE RESULT: ", result));
    },

    async handleImageAdded(file, Editor, cursorLocation) {
      const formData = new FormData();
      formData.append("image", file);

      const { data } = await axios({
        url: "https://api.imgur.com/3/image",
        method: "POST",
        headers: { Authorization: "Client-ID " + CLIENT_ID },
        data: formData,
      });
      console.log("TCL: handleImageAdded -> data", data);

      const { link, id } = data.data;
      Editor.insertEmbed(
        cursorLocation,
        "image",
        {
          id,
          url: link,
        },
        Quill.sources.USER
      );
    },

    handleImageRemoved(image) {
      console.log("handleImageRemoved -> image", image);
      this.deleteImage(image.id);
    },
    drag(bar, target, editor, preview, callback) {
      var getCss = function(o, key) {
        return o.currentStyle ? o.currentStyle[key] : document.defaultView.getComputedStyle(o, false)[key];
      };

      // console.log("owPreview: ", owPreview);
      var params = {
        left: 0,
        top: 0,
        currentX: 0,
        currentY: 0,
        flag: false,
      };
      if (getCss(target, "left") !== "auto") {
        params.left = getCss(target, "left");
      }
      if (getCss(target, "top") !== "auto") {
        params.top = getCss(target, "top");
      }
      let owEditor, owPreview;
      target.onmousedown = function(event) {
        owEditor = getCss(editor, "width");
        console.log("owEditor: ", owEditor);
        owPreview = getCss(preview, "width");
        params.flag = true;
        if (!event) {
          event = window.event;
          target.onselectstart = function() {
            return false;
          };
        }
        var e = event;
        params.currentX = e.clientX;
        params.currentY = e.clientY;
      };
      document.onmouseup = function() {
        params.flag = false;
        if (getCss(target, "left") !== "auto") {
          params.left = getCss(target, "left");
        }
        if (getCss(target, "top") !== "auto") {
          params.top = getCss(target, "top");
        }
      };
      document.onmousemove = function(event) {
        var e = event ? event : window.event;
        if (params.flag) {
          var nowX = e.clientX,
            nowY = e.clientY;
          var disX = nowX - params.currentX,
            disY = nowY - params.currentY;
          let owE = getCss(editor, "width");

          if (disX < 200 && disX > -200) {
            if (Number(owE.replace("px", "")) > 700 && disX > 0) {
              return;
            }
            if (Number(owE.replace("px", "")) < 300 && disX < 0) {
              return;
            }
            target.style.left = parseInt(params.left) + disX + "px";
            let width1 = Number(owEditor.replace("px", "")) + disX;
            let width2 = Number(owPreview.replace("px", "")) - disX;
            // console.log(width);
            editor.style.width = width1 + "px";
            preview.style.width = width2 + "px";
          }
          // target.style.left = parseInt(params.left) + disX + "px";
          // target.style.top = parseInt(params.top) + disY + "px";

          if (typeof callback == "function") {
            callback((parseInt(params.left) || 0) + disX, (parseInt(params.top) || 0) + disY);
          }
          if (event.preventDefault) {
            event.preventDefault();
          }
          return false;
        }
      };
    },
  },
};
</script>

<style lang="scss">
body {
  background-color: rgb(218, 226, 180);
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
  margin-top: 100px;
  padding: 0 200px;
  position: relative;

  .header {
    font-size: 36px;
    text-align: center;
    font-family: "STHeiti";
    line-height: 100px;
  }
  .button {
    position: absolute;
    top: 270px;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: rgb(23, 197, 14);
    left: 705px;
    cursor: pointer;
    transition: background-color 0.5s;
    &:hover {
      background-color: red;
    }
  }
  .vueEditor {
    float: left;
    width: 500px;

    .ql-toolbar {
      background-color: #fff;
      border-radius: 10px 10px 0px 0px;
    }
    .ql-container {
      background-color: #fff;
      border-radius: 0px 0px 10px 10px;
    }
  }
  .preview-wrap {
    float: left;
    width: 500px;
    margin-left: 20px;
    border: 1px solid #ccc;
    border-radius: 10px;
    background-color: #fff;
    .title {
      font-family: "STHeiti";
      padding: 10px;
      border: 1px solid #ccc;
      border-top: none;
      border-left: none;
      border-right: none;
      font-weight: bold;
      text-align: center;
    }
    .preview {
      margin-left: 20px;
      padding: 0 15px 0 0;
      word-break: break-all;
      min-height: 256px;
      word-wrap: break-word;

      .hljs-comment {
        word-break: break-all;

        word-wrap: break-word;
      }
      .ql-syntax {
        //  color:white;
        background-color: rgb(243, 240, 240);
        border-radius: 3px;
      }
    }
  }
}
</style>
