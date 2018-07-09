<template>
    <div class="medium-editor-container">
        <insert-image v-if="editor" class="insert-image-container" :editor="editor"></insert-image>
        <div class="editor" ref="editor"></div>
    </div>
</template>

<script>
import MediumEditor from 'medium-editor';
import InsertImage from './libs/InsertImage';
import _ from 'underscore';

export default {
  name: 'medium-editor',
  data() {
      return {
          editor: null,
          defaultOptions: {
          }
      }
  },
  props: [
      'options',
      'onChange'
  ],
  computed: {
      editorOptions () {
        return _.extend(this.defaultOptions, this.options);
      }
  },
  components: {
      InsertImage
  },
  mounted () {
      this.createElm();
  },
  methods: {
      createElm() {
        this.editor = new MediumEditor(this.$refs.editor, this.editorOptions)

        this.editor.subscribe('editableInput', () => {
            const content = this.editor.getContent()
            this.$emit('input', content)
            this.onChange(content)
        })
      },
      destroyElm() {
        this.editor.destroy()
      }
  },
  destroyed() {
      this.destroyElm()
  }
}
</script>

<style lang="css" scoped>
@import url('https://fonts.googleapis.com/css?family=Prompt');
* {
    outline: none;
    font-family: 'Prompt', sans-serif;
    box-sizing: border-box;
}
.medium-editor-container {
    max-width: 1000px;
    height: 100%;
    margin: 0 auto;
    position: relative;
    
}
</style>
