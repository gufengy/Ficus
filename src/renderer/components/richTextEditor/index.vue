<template>
  <div>
    <div id="vditor" class="vditor"/>
  </div>
</template>

<script>
import { onMounted, onBeforeUnmount, watch, nextTick } from 'vue'
import Vditor from 'ficus-editor'
import 'ficus-editor/dist/index.css'
import defineRAPI from './defineRAPI.js'
import bus from 'vue3-eventbus'

export default {
  name: 'TextUI',
  props: {
    content: {
      type: String,
      default: ''
    }
  },
  setup (props) {
    let vditor

    // 初始化
    function initVditor () {
      // options
      const options =
      {
        // 设置cdn
        cdn: '/ficus-editor',
        hint: {
          emojiPath: '/ficus-editor'
        },
        // 编辑器高度
        height: '100%',
        // 初始模式（所见即所得）
        mode: 'wysiwyg',
        // 缓存设置
        cache: {
          enable: false
        },
        // 设置回调延迟
        undoDelay: 50,
        // 预览模式选项
        preview: {
          // 设置预览最大宽度
          maxWidth: 896,
          // 预览时间间隔(毫秒)
          delay: 500,
          // 设备适配选项（暂时为空）
          actions: [],
          // 数学公式渲染选项
          math: {
            engine: 'KaTeX'
          },
          // 设置默认主题
          theme: {
            current: 'classic',
            path: '/ficus-editor/dist/css/content-theme'
          },
          // 设置markdown渲染选项
          markdown: {
            mark: true,
            linkBase: 'ficus://'
          }
        },
        // 设置tab键渲染
        tab: '\t',
        // 打字机模式（类似与typora）
        typewriterMode: false,
        // 内容为空时的提示
        placeholder: '请输入...',
        // 绑定快捷键
        ficusHotkey: [
          {
            hotkey: '⌘Z',
            action: () => {
              bus.emit('undoCurTab')
            }
          },
          {
            hotkey: '⇧⌘Z',
            action: () => {
              bus.emit('redoCurTab')
            }
          }
        ],
        // 创建实例后，将props中传入的内容展示出来, 并隐藏工具栏
        after: () => {
          vditor.setValue(props.content)
          vditor.hideToorBar()
        },
        // 用户输入回调函数，将最新的md字符串返回
        input: (content) => {
          bus.emit('saveChange', {
            content: content,
            wordCnt: content.length,
            lineCnt: content.split('\n').length - 1
          })
        },
        // 针对文件链接的回调函数
        link: {
          click: (aElement) => {
            if (aElement.getAttribute('class') === 'ficus-filelink') {
              const name = aElement.innerText
              let path = aElement.getAttribute('href')
              if (path.startsWith('ficus://')) {
                path = path.slice(8)
              }
              bus.emit('openRefFile', { name, path })
            } else {
              window.open(event.target.getAttribute('href'))
            }
          }
        }
      }
      // 根据options和默认设置创建vditor实例
      vditor = new Vditor('vditor', options)
    }
    // 监听传进来的值,set到编辑器里
    watch(
      () => props.content,
      (content) => {
        if (vditor) {
          vditor.setValue(content)
        }
      },
      {
        immediate: true
      }
    )
    // 初始化编辑器
    onMounted(() => {
      nextTick(() => {
        // 初始化vditor
        initVditor()
        // 定义和vditor相关的API, 使用全局事件总线实现
        defineRAPI(vditor)
      })
    })
    // 销毁
    onBeforeUnmount(() => {
      vditor.destroy()
      vditor = null
    })
  }
}
</script>

<style scoped>

</style>
