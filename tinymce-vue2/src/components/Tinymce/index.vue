<template>
  <div
    :class="{ fullscreen: fullscreen }"
    class="tinymce-container"
    :style="{ width: containerWidth }"
  >
    <textarea :id="tinymceId" class="tinymce-textarea" :key="tinymceFlag" />
    <div class="editor-custom-btn-container">
      <!-- <editorImage
        color="#1890ff"
        class="editor-upload-btn"
        @successCBK="imageSuccessCBK"
      /> -->
    </div>
  </div>
</template>

<script>
// import editorImage from "./components/EditorImage";
import plugins from './plugins';
import toolbar from './toolbar';
import load from './dynamicLoadScript';

// why use this cdn, detail see https://github.com/PanJiaChen/tinymce-all-in-one
// const tinymceCDN =
//   "https://cdn.jsdelivr.net/npm/tinymce@6.2.0/tinymce.min.js";

const tinymceCDN =
  'https://cdn.jsdelivr.net/npm/tinymce-all-in-one@4.9.3/tinymce.min.js';

export default {
  name: 'Tinymce',

  props: {
    id: {
      type: String,
      default: function () {
        return (
          'vue-tinymce-' +
          +new Date() +
          ((Math.random() * 1000).toFixed(0) + '')
        );
      },
    },
    value: {
      type: String,
      default: '',
    },
    toolbar: {
      type: Array,
      required: false,
      default() {
        return [];
      },
    },
    menubar: {
      type: String,
      default: 'file edit insert view format table',
    },
    height: {
      type: [Number, String],
      required: false,
      default: 360,
    },
    width: {
      type: [Number, String],
      required: false,
      default: 'auto',
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    // 上传图片函数
    uploadPicFn: {
      type: Function,
    },
  },
  data() {
    return {
      tinymceFlag: 1,
      hasChange: false,
      hasInit: false,
      tinymceId: this.id,
      fullscreen: false,
      languageTypeList: {
        en: 'en',
        zh: 'zh_CN',
        es: 'es_MX',
        ja: 'ja',
      },
      resourceUrl: {
        language_url:
          'https://cdn.jsdelivr.net/npm/tinymce-all-in-one@4.9.3/langs/zh_CN.js',
        skin_url:
          'https://cdn.jsdelivr.net/npm/tinymce-all-in-one@4.9.3/skins/lightgray',
        codesample_content_css:
          'https://cdn.jsdelivr.net/npm/tinymce-all-in-one@4.9.3/plugins/codesample/css/prism.css',
      },
    };
  },
  computed: {
    containerWidth() {
      const width = this.width;
      if (/^[\d]+(\.[\d]+)?$/.test(width)) {
        // matches `100`, `'100'`
        return `${width}px`;
      }
      return width;
    },
  },
  watch: {
    value: {
      immediate: true,
      handler(val) {
        if (!this.hasChange && this.hasInit) {
          this.$nextTick(() =>
            window.tinymce.get(this.tinymceId).setContent(val || '')
          );
        }
      },
    },
  },
  // components: { editorImage },
  mounted() {
    this.init();
  },
  activated() {
    if (window.tinymce) {
      this.initTinymce();
      this.tinymceFlag++;
    }
  },
  deactivated() {
    this.destroyTinymce();
    this.tinymceFlag++;
  },
  destroyed() {
    this.destroyTinymce();
  },
  methods: {
    init() {
      // dynamic load tinymce from cdn
      load(tinymceCDN, (err) => {
        if (err) {
          this.$message.error(err.message);
          return;
        }
        this.initTinymce();
      });
    },
    initTinymce() {
      const _this = this;
      window.tinymce.init({
        selector: `#${this.tinymceId}`,
        language_url: this.resourceUrl.language_url,
        skin_url: this.resourceUrl.skin_url,
        codesample_content_css: this.resourceUrl.codesample_content_css,
        language: this.languageTypeList['zh'],
        height: this.height,
        body_class: 'panel-body ',
        object_resizing: false,
        branding: false,
        toolbar_sticky: true,
        readonly: this.disabled ? 1 : 0, // 只读
        toolbar: this.disabled
          ? false
          : this.toolbar.length > 0
          ? this.toolbar
          : toolbar,
        // toolbar: "myCustomToolbarButton",
        menubar: this.menubar ? this.menubar : false,
        plugins: plugins,
        fontsize_formats: '8pt 10pt 11pt 12pt 14pt 16pt 18pt 24pt 36pt',
        font_formats:
          "微软雅黑='微软雅黑';宋体='宋体';黑体='黑体';仿宋='仿宋';楷体='楷体';隶书='隶书';幼圆='幼圆';Andale Mono=andale mono,times;Arial=arial,helvetica,sans-serif;Arial Black=arial black,avant garde;Book Antiqua=book antiqua,palatino;Comic Sans MS=comic sans ms,sans-serif;Courier New=courier new,courier;Georgia=georgia,palatino;Helvetica=helvetica;Impact=impact,chicago;Symbol=symbol;Tahoma=tahoma,arial,helvetica,sans-serif;Terminal=terminal,monaco;Times New Roman=times new roman,times;Trebuchet MS=trebuchet ms,geneva;Verdana=verdana,geneva;Webdings=webdings;Wingdings=wingdings",
        end_container_on_empty_block: true,
        powerpaste_word_import: 'clean',
        code_dialog_height: 450,
        code_dialog_width: 1000,
        // advlist_bullet_styles: "square",  // 设置项目符号
        // advlist_number_styles: "default", // 设置编号列表
        imagetools_cors_hosts: ['www.tinymce.com', 'codepen.io'],
        default_link_target: '_blank',
        target_list: false, // 隐藏插入链接中打开方式选项, 也可以自定义
        image_description: false, // 显示隐藏图片说明input
        link_title: false,
        nonbreaking_force_tab: true, // inserting nonbreaking space &nbsp; need Nonbreaking Space Plugin
        init_instance_callback: (editor) => {
          if (_this.value) {
            editor.setContent(_this.value);
          }
          _this.hasInit = true;
          editor.on('NodeChange Change KeyUp SetContent', () => {
            this.hasChange = true;
            this.$emit('input', editor.getContent());
          });
          // 监听粘贴事件
          editor.on('paste', (evt) => {
            // console.log("监听粘贴事件", evt);
            this.onPaste(evt);
          });
        },
        setup(editor) {
          editor.addButton('myCustomToolbarButton', {
            text: '插入兑换码',
            title: '插入兑换码',
            // icon: "el-icon-edit",
            // type: "menubutton",
            onclick: () => {
              editor.insertContent('{{redeem_code}}');
            },
          }),
            // menu: [
            //   {
            //     text: "Item 1",
            //     onclick: () => {
            //       editor.insertContent("Hello world!!");
            //     },
            //   },
            //   {
            //     text: "Item 2",
            //   },
            // ],
            // });
            editor.on('FullscreenStateChanged', (e) => {
              _this.fullscreen = e.state;
            });
        },

        // it will try to keep these URLs intact
        // https://www.tiny.cloud/docs-3x/reference/configuration/Configuration3x@convert_urls/
        // https://stackoverflow.com/questions/5196205/disable-tinymce-absolute-to-relative-url-conversions
        convert_urls: false,
        // 整合七牛上传
        // images_dataimg_filter(img) {
        //   setTimeout(() => {
        //     const $image = $(img);
        //     $image.removeAttr('width');
        //     $image.removeAttr('height');
        //     if ($image[0].height && $image[0].width) {
        //       $image.attr('data-wscntype', 'image');
        //       $image.attr('data-wscnh', $image[0].height);
        //       $image.attr('data-wscnw', $image[0].width);
        //       $image.addClass('wscnph');
        //     }
        //   }, 0);
        //   return img
        // },
        async images_upload_handler(blobInfo, success, failure, progress) {
          // console.log("自己上传", blobInfo.blob());
          progress(0);
          let file = blobInfo.blob();
          const res = await _this.uploadPicFn(file);
          success(res.cdn_url);
          progress(100);
          // const token = _this.$store.getters.token;
          // getToken(token).then(response => {
          //   const url = response.data.qiniu_url;
          //   const formData = new FormData();
          //   formData.append('token', response.data.qiniu_token);
          //   formData.append('key', response.data.qiniu_key);
          //   formData.append('file', blobInfo.blob(), url);
          //   upload(formData).then(() => {
          //     success(url);
          //     progress(100);
          //   })
          // }).catch(err => {
          //   failure('出现未知问题，刷新页面，或者联系程序员')
          //   console.log(err);
          // });
        },
      });
    },
    async onPaste(event) {
      // let self = this;

      // 实现图⽚粘贴上传
      const items = (event.clipboardData || window.clipboardData).items;
      // items为伪数组微信/QQ截图以及此富⽂本区域内复制粘贴的length为1,⿏标右键复制粘贴图⽚以及⽂本的复制粘贴的length为2;
      let len = items.length;
      for (let i = 0; i < len; i++) {
        if (items[i].kind == 'file' && items[i].type.indexOf('image') != -1) {
          // 判断是否为图⽚类型
          event.preventDefault(); // 如果是图⽚阻⽌⾃动粘贴到富⽂本编辑器
          let file = items[i].getAsFile(); // 获取⽂件数据

          // let blob = new Blob([file], { type: file.type }); //实例化⼀个blob 将图⽚⼤⼩以及类型初始化到blob⾥
          // let index = file.name.lastIndexOf(".");
          // let fileName =
          //   Date.now() + file.name.substring(index, file.name.length);
          // fileName --- 图⽚名称可⾃⾏处理
          // let file = blobInfo.blob();
          // const isLt2M = file.size / 1024 < 500;
          // if (!isLt2M) {
          //   failure("上传失败，图片不可超过500KB!");
          //   return false;
          // }
          // const formdate = new FormData();
          // formdate.append("file", file); //imageFile文件名和后端统一

          const res = await this.uploadPicFn(file);

          // console.log("res :>> ", res);
          // 插入网络链接图片
          window.tinymce.execCommand(
            'mceReplaceContent',
            false,
            `<img class="wscnph" src="${res.cdn_url}" >`
          );
        } else {
          // 不是图⽚类型直接粘贴, 跳过oss上传处理
          // console.log("粘贴的不是图⽚");
        }
      }
    },
    destroyTinymce() {
      const tinymce = window.tinymce.get(this.tinymceId);
      if (this.fullscreen) {
        tinymce.execCommand('mceFullScreen');
      }

      if (tinymce) {
        tinymce.destroy();
      }
    },
    setContent(value) {
      window.tinymce.get(this.tinymceId).setContent(value);
    },
    getContent() {
      window.tinymce.get(this.tinymceId).getContent();
    },
    imageSuccessCBK(arr) {
      arr.forEach((v) =>
        window.tinymce
          .get(this.tinymceId)
          .insertContent(`<img class="wscnph" src="${v.url}" >`)
      );
    },
  },
};
</script>
<!-- 
<style lang="scss" scoped>
  .tinymce-container {
    position: relative;
    line-height: normal;
  }

  .tinymce-container {
    ::v-deep {
      .mce-fullscreen {
        z-index: 10000;
      }
    }
  }

  .tinymce-textarea {
    visibility: hidden;
    z-index: -1;
  }

  .editor-custom-btn-container {
    position: absolute;
    right: 4px;
    top: 4px;
    /*z-index: 2005;*/
  }

  .fullscreen .editor-custom-btn-container {
    z-index: 10000;
    position: fixed;
  }

  .editor-upload-btn {
    display: inline-block;
  }
</style> -->
