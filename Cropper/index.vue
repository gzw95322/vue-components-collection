
<template>
  <div class="cp-content" :style="{'height': showUpload ? '140px' : '340px', 'paddingTop': showUpload ? '50px' : '0'}">
    <div class="cp-upload" v-show="showUpload">
        <label class="btn" for="uploads">本地上传</label>
        <input
          type="file"
          id="uploads"
          accept="image/png, image/jpeg, image/gif, image/jpg"
          @change="uploadImg($event, 1)"
        >
    </div>
    <vueCropper
      v-show="!showUpload"
      ref="cropper"
      :img="option.img"
      :outputSize="option.size"
      :outputType="option.outputType"
      :fixedBox="option.fixedBox"
      :canMove="option.canMove"
      :canMoveBox="option.canMoveBox"
      :autoCrop="option.autoCrop"
      :fixed="option.fixed"
      :original="option.original"
      :infoTrue="option.infoTrue"
      :full="option.full"
      :fixedNumber="option.fixedNumber" 
      :autoCropWidth="option.autoCropWidth"
      :autoCropHeight="option.autoCropHeight"
      :canScale="option.canScale"
      :centerBox="option.centerBox"
      @realTime="realTime"  
      :mode="option.mode"
    ></vueCropper>
  </div>
</template>
<script>
import { VueCropper } from "vue-cropper";

export default {
  props: {
    // 已有图片
    value: String,
    isfixed: Boolean
  },

  components: {
    VueCropper
  },

  data() {
    return {
        // showUpload: true,
      option: {
        img: this.value,
        size: 1,
        outputType: "jpg",
        outputSize: 1,
        fixedBox: false,
        autoCrop: true,
        canMove: true,
        canScale: true,
        canMoveBox: true,
        original: false,
        infoTrue: true,
        full: true,
        // 只有自动截图开启 宽度高度才生效
        fixed: !this.isfixed,
        fixedNumber: [1, 1],
        autoCrop: true,
        autoCropWidth: 284,
        autoCropHeight: 200,
        centerBox: false,
        cropData: {},
        mode: "contain"
      },
    };
  },

  methods: {
    uploadImg(e, num) {
      //上传图片
      // this.option.img
      var file = e.target.files[0];
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
        alert("图片类型必须是.gif,jpeg,jpg,png,bmp中的一种");
        return false;
      }
      var reader = new FileReader();
      reader.onload = e => {
        let data;
        if (typeof e.target.result === "object") {
          // 把Array Buffer转化为blob 如果是base64不需要
          data = window.URL.createObjectURL(new Blob([e.target.result]));
        } else {
          data = e.target.result;
        }
          this.option.img = data;
          // this.showUpload = false;
          this.$emit("input", data);
      };
      // 转化为base64
      reader.readAsDataURL(file)
      // 转化为blob
      // reader.readAsArrayBuffer(file);
      this.$refs.cropper.startCrop();
    },
    imgLoad(msg) {
      console.log(msg);
    },
    // 实时预览函数
    realTime(data) {
      //   this.previews = data;
      this.finish();
    },
    cropMoving(data) {
      this.option.cropData = data;
    },
    finish() {
      // 输出
      this.$refs.cropper.getCropData(data => {
          this.$emit("input", data);
        });
    },
    startCrop() {
      // start
      this.$refs.cropper.startCrop();
    },
    stopCrop() {
      //  stop
      this.$refs.cropper.stopCrop();
    }
  },
  computed: {
    showUpload() {
      return this.value == ''
    }
  }
};
</script> 

<style lang="less" scoped>
.cp-content {
  width: 520px;
  height: 340px;
  margin: 0 auto;
}
.cp-upload {
  text-align: center;
  .btn {
    cursor: pointer;
    position: relative;
    height: 50px;
    line-height: 50px;
    width: 160px;
    display: inline-block;
    color: #52ADDF;
    border: 2px #52ADDF solid;
    border-radius: 10px;
    font-size: 20px;
  }
  input {
    position: absolute;
    clip: rect(0 0 0 0);
  }
}
</style>