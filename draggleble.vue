<template>
  <div id="choose">
    <draggable v-model="currentImage" draggable=".picItem" class="flex" @start="onStart" @end="onEnd">
      <div v-for="(pic, index) in currentImage" :key="index" class="picItem">
        <div v-if="index === 0" class="mainPic">
          <div class="mainPicCover"></div>
          <div class="mainPicText">主图</div>
        </div>
        <div>
          <img style="object-fit: cover;" :src="pic" alt="pic" @click="handleImages(index)" />
        </div>
        <i @touchstart="removeImage(index)" @click="handleRemove(index)"></i>
      </div>
    </draggable>
  </div>
</template>

<style lang="less">
// css部分省略
</style>

<script>
import {
  ImagePreview
} from 'vant'
import draggable from 'vuedraggable'
import Vue from 'vue'

Vue.use(ImagePreview)

export default {
  props: {
    imgMaxLength: {
      type: Number,
      default: 5
    },
    images: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  data () {
    return {
      currentImage: [],
      diff: 0,
      isRmove: false,
      removeIndex: 0
    }
  },
  components: {
    draggable,
  },
  methods: {
    handleImages (index) {
      ImagePreview({
        images: this.currentImage,
        startPosition: index
      })
    },
    handleRemove (index) {
      this.$emit('removeImage', index)
    },
    removeImage (index) {
      this.isRmove = true
      this.removeIndex = index
    },
    // 通过start与end的时间差来模拟ios端的点击事件
    onStart () {
      this.diff = new Date().getTime()
    },
    onEnd (ele) {
      this.diff = new Date().getTime() - this.diff
      // 如果点击的过程小于200ms，说明是点击
      if (this.diff < 200) {
        // 说明点击的是关闭按钮
        if (this.isRmove) {
          this.isRmove = false
          setTimeout(() => {
            this.$emit('removeImage', this.removeIndex)
          }, 100);
        } else {
          // 进行预览
          ImagePreview({
            images: this.currentImage,
            startPosition: ele.oldIndex
          })
        }
        return
      }
      // 这儿说明是进行的拖动
      this.$emit('handleChangeImg', this.currentImage)
    },
  },
  watch: {
    images: {
      immediate: true,
      handler (val) {
        this.currentImage = [...val]
      }
    }
  }
}
</script>