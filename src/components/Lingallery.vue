<template>
  <div class="lingalleryContainer">
    <div :style="lingalleryStyle" class="lingallery">
      <large-view
        id="largeView"
        :accent-color="accentColor"
        :item="items[currentIndex]"
        v-if="addons.enableLargeView && showLargeView"
        @close-large-view="showLargeView = false"
      />
      <figure
        :style="figureStyle"
        itemprop="associatedMedia"
        itemscope
        itemtype="http://schema.org/ImageObject"
      >
        <div class="lingallery_spinner">
          <half-circle-spinner
            :animation-duration="1000"
            :color="accentColor"
            :size="60"
            v-if="isLoading"
          />
        </div>
        <video-element
          v-if="addons.enableVideoElement && items[currentIndex]['video'] !== undefined"
          :item="items[currentIndex]"
          :current-index="currentIndex"
        />
        <picture-element
          v-if="addons.enablePictureElement"
          :alt="currentAlt"
          :is-loading="isLoading"
          :main-image-style="mainImageStyle"
          :items="items"
          :current-index="currentIndex"
          ref="mainImage"
          @handle-large-image-click="handleLargeImageClick"
          @handle-image-swipe="handleImageSwipe"
          @handle-image-loaded="handleImageLoaded"
        />
        <img
          v-if="!addons.enablePictureElement && !(addons.enableVideoElement && items[currentIndex]['video'] !== undefined)"
          :alt="currentAlt"
          :class="{ loading: isLoading }"
          :src="currentImage"
          :style="mainImageStyle"
          @click="handleLargeImageClick"
          @load="handleImageLoaded"
          ref="mainImage"
          v-swipe="handleImageSwipe"
        />
        <div
          :style="captionStyle"
          class="lingallery_caption"
          v-if="currentCaption"
        >
          {{ currentCaption }}
        </div>
        <a
          @click="showPreviousImage"
          class="control left"
          v-if="!leftControlClass"
          ><span style="position:relative;top:calc(50% - 12px)"
            >&#9664;</span
          ></a
        >
        <a
          @click="showNextImage"
          class="control right"
          v-if="!rightControlClass"
          ><span style="position:relative;top:calc(50% - 12px)"
            >&#9654;</span
          ></a
        >
        <a
          :class="'control left ' + leftControlClass"
          @click="showPreviousImage"
          v-if="leftControlClass"
        ></a>
        <a
          :class="'control right ' + rightControlClass"
          @click="showNextImage"
          v-if="rightControlClass"
        ></a>
      </figure>
      <div class="lingallery_thumbnails" v-if="showThumbnails">
        <div class="lingallery_thumbnails_content">
          <div
            :key="index"
            class="lingallery_thumbnails_content_elem"
            v-for="(item, index) in items"
          >
            <img
              :alt="item.hasOwnProperty('alt') ? item.alt : ''"
              :src="item.thumbnail"
              :style="thumbnailStyle(index)"
              height="64"
              v-on="
                currentIndex !== index
                  ? { click: () => handleImageClick(index) }
                  : {}
              "
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import { HalfCircleSpinner } from 'epic-spinners'
import Hammer from 'hammerjs'
import Addons from './addons/index.js'

export default {
  name: 'Lingallery',
  mixins: [Addons],
  directives: {
    swipe: {
      bind: function(el, binding) {
        if (typeof binding.value === 'function') {
          const hammerjs = new Hammer(el)
          hammerjs.get('swipe').set({ direction: Hammer.DIRECTION_HORIZONTAL })
          hammerjs.on('swipe', binding.value)
        }
      }
    }
  },
  data() {
    return {
      currentImage: null,
      currentImageWidth: 0,
      currentImageHeight: 0,
      currentIndex: 0,
      currentId: null,
      currentCaption: '',
      currentAlt: '',
      windowWidth: 0,
      isLoading: false,
      showLargeView: false
    }
  },
  props: {
    items: {
      type: Array,
      default() {
        return []
      }
    },
    startImage: {
      type: Number,
      default: 0
    },
    width: {
      type: Number,
      default: 600
    },
    height: {
      type: Number,
      default: 400
    },
    baseColor: {
      type: String,
      default: '#fff'
    },
    accentColor: {
      type: String,
      default: '#3498db'
    },
    textColor: {
      type: String,
      default: '#000'
    },
    responsive: {
      type: Boolean,
      default: false
    },
    showThumbnails: {
      type: Boolean,
      default: true
    },
    mobileHeight: {
      type: Number,
      default: 0
    },
    mobileHeightBreakpoint: {
      type: Number,
      default: 0
    },
    leftControlClass: {
      type: String,
      default: ''
    },
    rightControlClass: {
      type: String,
      default: ''
    },
    disableImageClick: {
      type: Boolean,
      default: false
    },
    addons: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  components: {
    HalfCircleSpinner
  },
  computed: {
    lingalleryStyle() {
      return this.windowWidth > this.width && !this.responsive
        ? 'width:' + this.width + 'px'
        : 'width:100%'
    },
    captionStyle() {
      return 'color:' + this.textColor
    },
    mainImageStyle() {
      let isLoading =
        this.$refs.mainImage &&
        !this.addons.enablePictureElement &&
        this.$refs.mainImage.hasOwnProperty('classList') &&
        this.$refs.mainImage.classList.contains('loading')
          ? true
          : this.$refs.mainImage && this.addons.enablePictureElement
          ? this.$refs.mainImage.$el.children[
              this.$refs.mainImage.$el.children.length - 1
            ].classList.contains('loading')
          : false
      let mainImageStyle = ''

      if (
        this.mobileHeight !== 0 &&
        this.windowWidth < this.mobileHeightBreakpoint
      ) {
        mainImageStyle +=
          'width:100%;height:' + this.mobileHeight + 'px;object-fit:cover;'
      }
      if (isLoading) {
        if (this.mobileHeight !== 0) {
          mainImageStyle += 'min-height:' + this.mobileHeight + 'px;'
        } else mainImageStyle += 'min-height:200px;'
      }

      return mainImageStyle
    },
    figureStyle() {
      if (
        this.currentImage &&
        this.currentImageWidth &&
        this.currentImageHeight
      ) {
        let heightValue = 'height:auto'

        if (
          this.currentImageWidth < this.currentImageHeight &&
          this.mobileHeight === 0
        ) {
          heightValue = 'height:' + this.height + 'px'
        }
        return this.windowWidth > this.width && !this.responsive
          ? 'width:' + this.width + 'px;height:' + this.height + 'px'
          : 'width:100%;' + heightValue
      } else
        return this.windowWidth > this.width && !this.responsive
          ? 'width:' + this.width + 'px;height:' + this.height + 'px'
          : 'width:100%;height:auto'
    },
  },
  methods: {
    initLingallery() {
      this.currentImage = this.items[this.startImage].src;
      this.currentCaption = this.items[this.startImage].caption
      this.currentId = this.items[this.startImage].id
      this.currentIndex = this.startImage
      this.windowWidth = window.innerWidth
      this.sendId()
    },
    handleImageLoaded() {
      this.isLoading = false
      this.updateCurrentImageSizes()
    },
    updateCurrentImageSizes() {
      let img =
        this.$refs.mainImage && this.$refs.mainImage.hasOwnProperty('src')
          ? this.$refs.mainImage
          : this.$refs.mainImage.$el
          ? this.$refs.mainImage.$el.getElementsByTagName('img')[0]
          : null
      if (img) {
        this.currentImageWidth = img.naturalWidth
        this.currentImageHeight = img.naturalHeight
      }
    },
    handleImageSwipe(event) {
      if (event.direction === 4) {
        this.showPreviousImage()
      } else if (event.direction === 2) {
        this.showNextImage()
      }
    },
    handleImageClick(index) {
      this.currentIndex = index
      this.pickImage(index)
    },
    handleLargeImageClick() {
      if (this.addons.enableLargeView) {
        this.showLargeView = true
      } else if (!this.disableImageClick) {
        this.showNextImage()
      }
    },
    handleLoader(isLoading) {
      this.isLoading = isLoading
    },
    pickImage(index) {
      // Show Loader if is not a video
      if(!this.isVideo()) {
        this.handleLoader(true);
      }

      this.currentImage = this.items[index].src
      this.currentCaption = this.items[index].hasOwnProperty('caption')
        ? this.items[index].caption
        : ''
      this.currentAlt = this.items[index].hasOwnProperty('alt')
        ? this.items[index].alt
        : ''
      this.currentId = this.items[index].hasOwnProperty('id')
        ? this.items[index].id
        : ''

      this.sendId()
    },
    thumbnailStyle(index) {
      let color =
        this.currentIndex === index ? this.accentColor : this.baseColor
      return 'border-color:' + color
    },
    sendId() {
      this.$emit('update:iid', this.currentId)
    },
    showNextImage() {
      // Show Loader if is not a video
      if(!this.isVideo()) {
        this.handleLoader(true);
      }

      if (this.items.length > this.currentIndex + 1) {
        this.currentIndex = this.currentIndex + 1
      } else {
        this.currentIndex = 0
      }

      this.pickImage(this.currentIndex)
    },
    showPreviousImage() {
      // Show Loader if is not a video
      if(!this.isVideo()) {
        this.handleLoader(true);
      }

      if (this.currentIndex !== 0) {
        this.currentIndex = this.currentIndex - 1
      } else {
        this.currentIndex = this.items.length - 1
      }

      this.pickImage(this.currentIndex)
    },
    isVideo() {
      return this.addons.enableVideoElement
              && this.items[this.currentIndex]['video'] !== undefined
    }
  },
  created() {
    this.initAddons()
  },
  mounted() {
    this.initLingallery()
  },
  watch: {
    items() {
      this.currentImage = this.items[this.startImage].src
    }
  }
}
</script>

<style lang="scss" scoped>
.lingalleryContainer {
  /deep/ .lingallery {
    max-width: 100%;
    figure {
      position: relative;
      margin: 0;
      padding: 0;
      max-width: 100%;
      text-align: center;
      cursor: pointer;
      img {
        max-width: 100%;
        max-height: 100%;
        transition: opacity 0.25s ease;
        &.loading {
          opacity: 0.25;
          min-height: 200px;
          transition: opcacity 0.25s ease;
        }
      }
      a.control {
        position: absolute;
        top: 0;
        padding-left: 5px;
        padding-right: 15px;
        height: 100%;
        display: none;
        font-size: 20px;
        color: #fff;
        cursor: pointer;
        text-shadow: 0 0 20px rgba(0, 0, 0, 0.75);
        &:before {
          position: relative;
          top: calc(50% - 12px);
        }
        &.left {
          left: 0;
        }
        &.right {
          right: 0;
        }
      }
      &:hover {
        a.control {
          display: block;
        }
      }
    }
    .lingallery_caption {
      position: absolute;
      bottom: 0;
      left: 0;
      padding: 4px 0;
      width: 100%;
      background-color: rgba(255, 255, 255, 0.75);
      font-size: 1em;
    }
    .lingallery_thumbnails {
      overflow-x: auto;
      width: 100%;
      scroll-snap-type: x mandatory;
      &::-webkit-scrollbar {
        height: 6px;
      }
      &::-webkit-scrollbar-track {
        border-radius: 10px;
        background: #eaeaea;
      }
      &::-webkit-scrollbar-thumb {
        border-radius: 10px;
        background: #b4b4b4;
      }
      &::-webkit-scrollbar-thumb:window-inactive {
        background: rgba(100, 100, 100, 0.4);
      }
      .lingallery_thumbnails_content {
        margin-top: 2px;
        width: auto;
        white-space: nowrap;
        .lingallery_thumbnails_content_elem {
          display: inline-block;
          scroll-snap-align: start;
          img {
            border: 2px solid #fff;
            cursor: pointer;
          }
        }
      }
    }
    .lingallery_spinner {
      position: absolute;
      left: 50%;
      top: calc(50% - 32px);
      > div {
        z-index: 9999;
        position: relative;
        left: -50%;
      }
    }
  }
}
</style>
