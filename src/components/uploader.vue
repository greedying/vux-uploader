<template>
  <div class="weui-cell">
    <div class="weui-cell__bd">
      <div class="weui-uploader">
        <div v-show="showHeader" class="weui-uploader__hd">
          <p class="weui-uploader__title"> {{ title }} </p>
          <div v-show="showHeader && showTip" class="weui-uploader__info">
            {{ images.length }} / {{ max }}
          </div>
        </div>

        <div class="weui-uploader__bd" :class="{small: size === 'small'}">
          <div v-show="!readonly && images.length > 0" class="weui-uploader__input-box remove" @click="remove">
          </div>

          <ul class="weui-uploader__files">
            <uploader-item v-for="image in images" :background-image="image.url" :key="image.url" @click.native="preview(image)"></uploader-item>
          </ul>
          
          <div v-show="!readonly && images.length < max" class="weui-uploader__input-box" @click="add">
            <input v-if="!handleClick" ref="input" class="weui-uploader__input" type="file" accept="image/*" :capture="showCapture" @change="change">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UploaderItem from './uploader-item.vue'
import axios from 'axios'
export default {
  props: {
    images: {
      type: Array,
      default: () => []
    },
    max: {
      type: Number,
      default: 1
    },
    showHeader: {
      type: Boolean,
      default: true
    },
    showTip: {
      type: Boolean,
      default: true
    },
    readonly: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: '图片上传'
    },
    // 是否接管+号的click事件，如果是，点击不弹出选择文件的框
    handleClick: {
      type: Boolean,
      default: false
    },
    // 选择文件后是否自动上传，如果不是，则emit change事件
    autoUpload: {
      type: Boolean,
      default: true
    },
    uploadUrl: {
      type: String,
      default: ''
    },
    size: {
      type: String,
      default: 'normal'
    },
    capture: {
      type: String,
      default: ''
    },
    name: {
      type: String,
      default: 'img'
    },
    params: {
      type: Object,
      default: null
    }
  },
  components: {
    UploaderItem
  },
  methods: {
    add () {
      if (this.handleClick) {
        this.$emit('add-image')
      }
    },
    // 移除第一张图
    remove () {
      if (this.handleClick) {
        this.$emit('remove-image')
      } else {
        this.images.shift()
      }
    },
    preview (image) {
      // 暂未实现，可以捕捉preview事件自己实现
      this.$emit('preview', image)
    },
    // 适用于action的情况
    change () {
      if (this.handleClick) {
        return
      }

      let formData = new window.FormData()
      formData.append(this.name, this.$refs.input.files[0])
      if (this.params) {
        for( let key in this.params) {
          formData.append(key, this.params[key])
        }
      }

      if (this.autoUpload) {
        if (!this.uploadUrl) {
          console.error('uploadUrl不应为空')
        }

        if (this.$vux && this.$vux.loading) {
          this.$vux.loading.show('正在上传')
        }

        axios.post(this.uploadUrl, formData)
        .then((response) => {
          if (this.$vux && this.$vux.loading) {
            this.$vux.loading.hide()
          }
          this.$refs.input.value = ''
          this.images.push(response.data.data)
        })
      } else {
        this.$emit('upload-image', formData)
      }
    }
  },
  computed: {
    showCapture () {
      return this.capture || undefined
    }
  }
}
</script>
<style scoped lang="less">
@import '~vux/src/styles/weui/widget/weui-uploader/index.less';
.remove:before {
  width: 0;
  height: 0;
}
.weui-uploader__bd.small {
  .weui-uploader__input-box {
    margin-right: 5px;
    margin-bottom: 5px;
    width: 58px;
    height: 58px;
  }
  .weui-uploader__file {
    width: 60px;
    height: 60px;
    margin-right: 5px;
    margin-bottom: 5px;
  }
}
</style>
