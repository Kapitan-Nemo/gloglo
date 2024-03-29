<script>
import { Editor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import TextAlign from '@tiptap/extension-text-align'
import Underline from '@tiptap/extension-underline'
import CharacterCount from '@tiptap/extension-character-count'
import Image from '@tiptap/extension-image'

export default {
  components: {
    EditorContent,
  },
  props: {
    modelValue: {
      type: String,
      default: '',
    },
    maxLimit: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      editor: null,
      textActions: [
        { slug: 'image', icon: 'ri-image-add-line', active: 'image' },
        { slug: 'bold', icon: 'ri-bold', active: 'bold' },
        { slug: 'italic', icon: 'ri-italic', active: 'italic' },
        { slug: 'underline', icon: 'ri-underline', active: 'underline' },
        { slug: 'strike', icon: 'ri-strikethrough', active: 'strike' },
        {
          slug: 'align',
          option: 'left',
          icon: 'ri-align-left',
          active: { textAlign: 'left' },
        },
        {
          slug: 'align',
          option: 'center',
          icon: 'ri-align-center',
          active: { textAlign: 'center' },
        },
        {
          slug: 'align',
          option: 'right',
          icon: 'ri-align-right',
          active: { textAlign: 'right' },
        },
        {
          slug: 'align',
          option: 'justify',
          icon: 'ri-align-justify',
          active: { textAlign: 'justify' },
        },
        { slug: 'bulletList', icon: 'ri-list-unordered', active: 'bulletList' },
        { slug: 'orderedList', icon: 'ri-list-ordered', active: 'orderedList' },
        { slug: 'undo', icon: 'ri-arrow-go-back-line', active: 'undo' },
        { slug: 'redo', icon: 'ri-arrow-go-forward-line', active: 'redo' },
        { slug: 'clear', icon: 'ri-format-clear', active: 'clear' },
        { slug: 'code', icon: 'ri-code-view', active: 'code' },
      ],

    }
  },
  computed: {
    charactersCount() {
      return this.editor.storage.characterCount.characters()
    },
    wordsCount() {
      return this.editor.storage.characterCount.words()
    },
    limitWarning() {
      const isCloseToMax = this.charactersCount >= this.maxLimit - 20
      const isMax = this.charactersCount === this.maxLimit

      if (isCloseToMax && !isMax)
        return 'warning'
      if (isMax)
        return 'danger'

      return ''
    },
  },
  watch: {
    modelValue(value) {
      if (this.editor.getHTML() === value)
        return
      this.editor.commands.setContent(this.modelValue, false)
    },
  },
  mounted() {
    this.editor = new Editor({
      content: this.modelValue,
      extensions: [
        StarterKit,
        Image,
        Underline,
        CharacterCount.configure({
          limit: this.maxLimit,
        }),
        TextAlign.configure({
          types: ['heading', 'paragraph'],
        }),
      ],
      onUpdate: () => {
        this.$emit('update:modelValue', this.editor.getHTML())
      },
    })
  },
  beforeUnmount() {
    this.editor.destroy()
  },
  methods: {
    addImage() {
      const url = window.prompt('URL')
      if (url)
        this.editor.chain().focus().setImage({ src: url }).run()
    },
    onActionClick(slug, option = null) {
      const vm = this.editor.chain().focus()
      const actionTriggers = {
        bold: () => vm.toggleBold().run(),
        italic: () => vm.toggleItalic().run(),
        underline: () => vm.toggleUnderline().run(),
        strike: () => vm.toggleStrike().run(),
        bulletList: () => vm.toggleBulletList().run(),
        orderedList: () => vm.toggleOrderedList().run(),
        align: () => vm.setTextAlign(option).run(),
        undo: () => vm.undo().run(),
        redo: () => vm.redo().run(),
        clear: () => {
          vm.clearNodes().run()
          vm.unsetAllMarks().run()
        },
        code: () => vm.toggleCodeBlock().run(),
        image: () => this.addImage(),
      }

      actionTriggers[slug]()
    },
    onHeadingClick(index) {
      const vm = this.editor.chain().focus()
      vm.toggleHeading({ level: index }).run()
    },
  },
}
</script>

<template>
  <div id="text-editor">
    <div v-if="editor" class="toolbar">
      <div class="align-dropdown">
        <p class="dropbtn">
          Heading ▼
        </p>
        <div class="dropdown-content">
          <a
            v-for="index in 6"
            :class="{ active: editor.isActive('heading', { level: index }) }"
            :style="{ fontSize: `${20 - index}px` }"
            role="button"
            @click="onHeadingClick(index)"
          >
            H{{ index }}
          </a>
        </div>
      </div>

      <button
        v-for="({ slug, option, active, icon }, index) in textActions"
        :class="{ active: editor.isActive(active) }"
        @click="onActionClick(slug, option)"
      >
        <i :class="icon" />
      </button>
    </div>

    <EditorContent :editor="editor" />

    <div v-if="editor" class="footer">
      <span class="characters-count" :class="maxLimit ? limitWarning : ''">
        {{ charactersCount }}
        {{ maxLimit ? `/ ${maxLimit} characters` : 'characters' }}
      </span>
      |
      <span class="words-count"> {{ wordsCount }} words </span>
    </div>
  </div>
</template>

<style lang="scss" scoped>
#text-editor {
  border: 1px solid #808080;

  .toolbar {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 1px solid #808080;

    > button {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 32px;
      height: 32px;
      font-size: 20px;
      background: #fff;
      color: #333;
      border: none;
      border-radius: 2px;
      margin: 0.5em 4px;
      -webkit-appearance: none;
      appearance: none;

      cursor: pointer;

      &.active {
        background: #333;
        color: #fff;
      }
    }
  }

  .align-dropdown {
    position: relative;
    display: inline-block;
    margin: 0.5em 8px;

    > button {
      height: 32px;
      background: #fff;
      color: #333;
      border: none;
      border-radius: 2px;
      -webkit-appearance: none;
      appearance: none;
      cursor: pointer;
    }

    > .dropdown-content {
      display: none;
      position: absolute;
      left: 0;
      right: 0;
      border: 1px solid #333;
      outline: 1px solid #fff;
      border-radius: 2px;
      background-color: #fff;
      z-index: 1;
      color: black;

      a {
        display: block;
        padding: 6px 12px;
        text-align: center;
        cursor: pointer;

        &:hover,
        &.active {
          background: #333;
          color: #fff;
        }
      }
    }

    &:hover .dropdown-content {
      display: block;
    }
  }

  .divider {
    width: 1px;
    height: 24px;
    background: #333;
    margin-right: 6px;
  }

  .footer {
    color: #808080;
    font-size: 14px;
    text-align: right;
    padding: 6px;

    .characters-count {
      &.warning {
        color: orange;
      }

      &.danger {
        color: red;
      }
    }
  }

  .ProseMirror {
    height: 300px;
    overflow-y: auto;
    padding-left: 0.5em;
    padding-right: 0.5em;
    outline: none;

    > p:first-child {
      margin-top: 0.5em;
    }

    > h1,
    h2,
    h3,
    h4,
    h5,
    h6 {
      &:first-child {
        margin-top: 0.5em;
      }
    }
  }
}
</style>
