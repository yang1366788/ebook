<template>
  <div class="ebook">
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
   <div class="read-wrapper">
    <div id="read"></div>
    <div class="mask">
      <div class="left" @click="prevPage"></div>
      <div class="center" @click="toggleTitleAndMenu"></div>
      <div class="right" @click="nextPage"></div>
    </div>
   </div>
   <menu-bar @setFontSize="setFontSize" :ifTitleAndMenuShow="ifTitleAndMenuShow" :defaultFontSize="defaultFontSize" :fontSizeList="fontSizeList" ref="menubar"
   :themeList="themeList"
   :defaultTheme="defaultTheme"
   @setTheme="setTheme"
   :bookAvailable="bookAvailable"
   @onProgressChange="onProgressChange"
   :navigation="navigation"
   @jumpTo="jumpTo"
   ></menu-bar>
  </div>
</template>

<script>
import TitleBar from '@/components/TitleBar'
import MenuBar from '@/components/MenuBar'
import Epub from 'epubjs'
const DOWNLOAD_URL = '/static/book2.epub'
export default {
  data () {
    return {
      ifTitleAndMenuShow: false,
      fontSizeList: [
        {fontSize: 12},
        {fontSize: 14},
        {fontSize: 16},
        {fontSize: 18},
        {fontSize: 20},
        {fontSize: 22},
        {fontSize: 24}
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: 'default',
          style: {
            body: {
              'color': '#000',
              'background': '#fff'
            }
          }
        },
        {
          name: 'eye',
          style: {
            body: {
              'color': '#000',
              'background': '#ceeaba'
            }
          }
        },
        {
          name: 'night',
          style: {
            body: {
              'color': '#fff',
              'background': '#000'
            }
          }
        },
        {
          name: 'gold',
          style: {
            body: {
              'color': '#000',
              'background': 'rgb(241,236,226)'
            }
          }
        }
      ],
      defaultTheme: 0,
      bookAvailable: false,
      navigation: {}
    }
  },
  components: {
    TitleBar,
    MenuBar
  },
  methods: {
    jumpTo (href) {
      this.rendition.display(href)
      this.hideTitleAndMenu()
    },
    hideTitleAndMenu () {
      this.ifTitleAndMenuShow = false
      this.$refs.menubar.hiddenSetting()
      this.$refs.menubar.hideContent()
    },
    onProgressChange (progress) {
      const precentage = progress / 100
      const location = precentage > 0 ? this.locations.cfiFromPercentage(precentage) : 0
      this.rendition.display(location)
    },
    setTheme (index) {
      this.themes.select(this.themeList[index].name)
      this.defaultTheme = index
    },
    registerTheme () {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style)
      })
    },
    setFontSize (fontSize) {
      this.defaultFontSize = fontSize
      if (this.themes) {
        this.themes.fontSize(fontSize + 'px')
      }
    },
    toggleTitleAndMenu () {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menubar.hiddenSetting()
      }
    },
    prevPage () {
      // rendition.prev
      if (this.rendition) {
        this.rendition.prev()
      }
    },
    nextPage () {
      // rendition.next
      if (this.rendition) {
        this.rendition.next()
      }
    },
    // 电子书的解析和渲染
    showEnpub () {
      // 生成book对象
      this.book = new Epub(DOWNLOAD_URL)
      // 生成Rendition,通过book.renderTo
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight
      })
      // 通过Rendition.display渲染
      this.rendition.display()
      // 获取theme对象
      this.themes = this.rendition.themes
      // this.themes.register(name,style)
      // this.themes.select(name)
      this.setFontSize(this.defaultFontSize)// 设置默认字体
      this.registerTheme()
      this.setTheme(this.defaultTheme)
      // 获取locations对象
      // 通过epubjs的钩子函数实现
      this.book.ready.then(() => {
        this.navigation = this.book.navigation
        return this.book.locations.generate()
      }).then(result => {
        this.locations = this.book.locations
        this.bookAvailable = true
      })
    }
  },
  mounted () {
    this.showEnpub()
  }
}
</script>
<style lang='scss' scoped>
@import 'assets/style/global';
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
