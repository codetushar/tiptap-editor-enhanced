<template>
  <div class="editor-container">
    <div class="toolbar">
      <div class="toolbar-group">
        <button
          v-for="actionName in activeButtons"
          :key="actionName"
          class="toolbar-button"
          :class="{
            'is-active': isButtonActive(actionName),
            bold: actionName === 'bold',
            italic: actionName === 'italic',
            strike: actionName === 'strike',
            list: ['bulletList', 'orderedList'].includes(actionName),
            'undo-redo': ['undo', 'redo'].includes(actionName)
          }"
          @click="handleButtonClick(actionName)"
        >
          <span class="font-input-weight" v-if="actionName === 'font-weight'">
            <input
              type="range"
              min="1"
              max="1000"
              v-model="fontWeight"
              v-if="showFontSlider"
              @input="updateFontWeight"
            />
            <span class="current-weight">{{ fontWeight }}</span>
          </span>
          <span v-else>{{ getButtonLabel(actionName) }}</span>
        </button>
        <button class="toolbar-button" @click="downloadFile">Download</button>
      </div>
    </div>
    <editor-content class="editor-content" :editor="editor" />
  </div>
</template>

<script>
import { Editor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import TextStyle from '@tiptap/extension-text-style'
import { FontWeight } from './FontWeight'

export default {
  name: 'TipTap',
  components: {
    EditorContent
  },
  props: {
    initialContent: {
      type: String,
      required: true,
      default: '<span>editable text</span>'
    },
    activeButtons: {
      type: Array,
      validator: function (list) {
        for (let el of list) {
          if (
            [
              'bold',
              'italic',
              'strike',
              'font-weight',
              'bulletList',
              'orderedList',
              'undo',
              'redo'
            ].indexOf(el) === -1
          ) {
            return -1
          }
        }
        return 1
      },
      default: () => [
        'bold',
        'italic',
        'strike',
        'font-weight',
        'bulletList',
        'orderedList',
        'undo',
        'redo'
      ]
    }
  },
  emits: ['update'],
  data() {
    return {
      editor: null,
      fontWeight: 100,
      showFontSlider: false
    }
  },
  watch: {
    fontWeight(newWeight) {
      this.editor.chain().setFontWeight(`${newWeight}`).focus().run()
      if (!this.isWordSelection()) {
        this.editor.chain().unsetFontWeight().focus().run()
      }
    }
  },
  created() {
    this.editor = new Editor({
      content: this.initialContent,
      extensions: [StarterKit, TextStyle, FontWeight]
    })

    this.editor.on('update', () => {
      this.$emit('update', this.editor.getHTML())
    })
    this.editor.on('selectionUpdate', () => {
      if (this.isWordSelection()) {
        this.showFontSlider = true
      } else {
        this.showFontSlider = false
      }
    })
  },
  beforeUnmount() {
    this.editor.destroy()
  },
  methods: {
    isWordSelection() {
      const text = this.editor.state.doc.textBetween(
        this.editor.state.selection.from,
        this.editor.state.selection.to
      )

      return text.length
    },
    isButtonActive(actionName) {
      switch (actionName) {
        case 'bold':
          return this.editor.isActive('bold')
        case 'italic':
          return this.editor.isActive('italic')
        case 'strike':
          return this.editor.isActive('strike')
        case 'bulletList':
          return this.editor.isActive('bulletList')
        case 'orderedList':
          return this.editor.isActive('orderedList')
        default:
          return false
      }
    },
    handleButtonClick(actionName) {
      switch (actionName) {
        case 'bold':
          this.editor.chain().focus().toggleBold().run()
          break
        case 'italic':
          this.editor.chain().focus().toggleItalic().run()
          break
        case 'strike':
          this.editor.chain().focus().toggleStrike().run()
          break
        case 'bulletList':
          this.editor.chain().focus().toggleBulletList().run()
          break
        case 'orderedList':
          this.editor.chain().focus().toggleOrderedList().run()
          break
        case 'undo':
          this.editor.chain().focus().undo().run()
          break
        case 'redo':
          this.editor.chain().focus().redo().run()
          break
      }
    },
    getButtonLabel(actionName) {
      switch (actionName) {
        case 'bold':
          return 'B'
        case 'italic':
          return 'I'
        case 'strike':
          return 'S'
        case 'bulletList':
          return '•'
        case 'orderedList':
          return '1.'
        case 'undo':
          return '⟲'
        case 'redo':
          return '⟳'
      }
    },
    downloadFile() {
      const filename = 'my-tiptap-content.txt'
      const contentType = 'text/html'
      const json = this.editor.getJSON()
      const jsonText = processJson(json)
      const content = this.editor.getText()
      console.log(json)
      downloadContent(filename, contentType, jsonText)
    },
    extractFontWeight(styleString) {
      const fontWeightRegex = /font-weight:\s*(\d+)/
      const match = fontWeightRegex.exec(styleString)
      return match ? match[1] : 'normal' // Default if not found
    }
  }
}

function processJson(data) {
  let outputText = ''
  const wordPattern = /\b\w+\b/g

  if (data.type === 'text') {
    const fontWeight = data?.marks?.find(
      (style) => style.type === 'textStyle' && style?.attrs?.fontWeight
    ).attrs?.fontWeight
    const text = data.text.replace(wordPattern, (word) => {
      if (fontWeight) {
        return word + ':' + fontWeight
      }
      return word
    })
    outputText += text
  }
  if (data?.content?.length > 0) {
    for (const item of data.content) {
      outputText += processJson(item)
    }
  }

  if (data.type === 'paragraph') outputText += '\n'

  return outputText
}
function downloadContent(filename, contentType, content) {
  // Create a temporary link element
  const temporaryLink = document.createElement('a')

  // Construct a data URI based on your content type
  const dataURI = `data:${contentType};charset=utf-8,` + encodeURIComponent(content)

  temporaryLink.href = dataURI // Set the href for download
  temporaryLink.download = filename // Set the filename

  document.body.appendChild(temporaryLink) // Required for Firefox
  temporaryLink.click()
  document.body.removeChild(temporaryLink)
}
</script>

<style scoped>
.editor-container {
  display: flex;
  flex-direction: column;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.toolbar {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 8px;
  background-color: #f7f7f7;
  border-bottom: 1px solid #ccc;
}

.toolbar-group {
  display: flex;
  align-items: center;
}

.toolbar-button {
  display: flex;
  justify-content: center;
  align-items: center;
  min-width: 32px;
  height: 32px;
  margin-right: 8px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  color: #333;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}

.toolbar-button:hover {
  background-color: #eee;
}

.toolbar-button.is-active {
  background-color: #ddd;
  font-weight: bold;
}

.toolbar-button.bold {
  font-weight: bold;
}

.toolbar-button.italic {
  font-style: italic;
}

.toolbar-button.strike {
  text-decoration: line-through;
}

.toolbar-button.list {
  font-weight: bold;
}

.toolbar-button.undo-redo {
  font-weight: bold;
}

.current-weight {
  margin-left: 4px;
  font-size: 12px;
  color: #666;
}

.font-input-weight {
  margin-left: 4px;
  color: #666;
}

.editor-content {
  flex: 1;
  padding: 16px;
  font-size: 16px;
  line-height: 1.5;
}
</style>
