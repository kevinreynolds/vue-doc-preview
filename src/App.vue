<script>
import axios from 'axios'
import Markdown from './components/Markdown'
import TextPreview from './components/TextPreview'
import Office from './components/Office'

export default {
  name: 'app',
  components: {
    Markdown,
    TextPreview,
    Office
  },
  props: {
    type: {
      type: String,
      default: 'md'
    },
    value: {
      type: String,
      default: ''
    },
    height: {
      type: Number,
      default: 90
    },
    language: {
      type: String,
      default: ''
    },
    mdStyle: {
      type: Object,
      default: null
    },
    textStyle: {
      type: Object,
      default: null
    },
    url: {
      type: String,
      default: ''
    }
  },
  data: function () {
    return {
      tempValue: '',
      parseComponet: 'Markdown',
      styler: ''
    }
  },
  watch: {
    type: function (val) {
      this.updateType()
    }
  },
  computed: {
    actualValue: function () {
      this.tempValueC()
      if (this.type === 'office') {
        return `//view.officeapps.live.com/op/view.aspx?src=${this.tempValue}`
      } else if (this.type === 'code' && this.language) {
        return `\`\`\`${this.language}\n${this.tempValue}\n\`\`\``
      } else {
        return this.tempValue
      }
    }
  },
  mounted: function () {
    this.updateType()
  },
  methods: {
    tempValueC: function () {
      if (this.value) {
        this.tempValue = this.value
      } else {
        if (this.type !== 'office') {
          let self = this
          this.download(this.url).then(data => {
            self.tempValue = data
          }).catch(err => {
            self.tempValue = 'Download Error!'
            console.error(err)
          })
        } else {
          this.tempValue = this.url
        }
      }
    },
    download: function (url, onProgress) {
      return new Promise((resolve, reject) => {
        axios.get(url, {
          responseType: 'blob',
          onDownloadProgress: function (progressEvent) {
            if (onProgress && typeof onProgress === 'function') {
              onProgress(progressEvent) // progress callback
            }
          }
        }).then(res => {
          const reader = new FileReader()
          reader.readAsText(new Blob([res.data]))
          reader.onload = function () {
            resolve(reader.result)
          }
        }).catch(err => {
          reject(err)
        })
      })
    },
    setHeiht: function () {
      let height = this.height
      if (height < 0) height = 0
      if (height > 100) {
        // height大于100时将视作固定高度，单位px
        this.styler = `height: ${height}px`
      } else {
        // height小于等于100时为百分比高度
        if (this.type === 'office') {
          const contentHeight = this.getClientHeight() * height / 100
          this.styler = `height: ${contentHeight}px`
        } else {
          this.styler = `height: ${height}%`
        }
      }
    },
    getClientHeight: function () {
      const clientHeight = document.documentElement.clientHeight
      return clientHeight
    },
    updateType: function () {
      switch (this.type) {
        case 'markdown':
          this.parseComponet = 'Markdown'
          break
        case 'text':
          this.parseComponet = 'TextPreview'
          break
        case 'office':
          this.parseComponet = 'Office'
          break
        case 'code':
          this.parseComponet = 'Markdown'
          break
      }
      this.setHeiht()
    }
  },
  render: function (h) {
    let component = <div>Prop type error, Type must be markdown/office/text/code!</div>
    if (this.type === 'markdown' || this.type === 'code') {
      component = <Markdown value={this.actualValue} mdStyle={this.mdStyle} />
    } else if (this.type === 'text') {
      component = <TextPreview value={this.actualValue} textStyle={this.textStyle} />
    } else if (this.type === 'office') {
      component = <Office value={this.actualValue} />
    }
    return (
      <div
        id="VueDocPreviewRoot"
        class="root"
        style={this.styler}>
        {component}
      </div>
    )
  }
}
</script>

<style lang="less" scoped>
  .root {
    height: 100%;
  }
</style>
