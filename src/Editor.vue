<template>
    <div class="medium-editor-container">
        <insert-image v-if="editor" :uploadUrl="options.uploadUrl" :editorRef="$refs.editor" :editor="editor"></insert-image>
        <list-handler v-if="editor" :editor="editor"></list-handler>
        <div class="editor" ref="editor"></div>
    </div>
</template>

<script>
import MediumEditor from 'medium-editor';
import InsertImage from './libs/InsertImage';
import ListHandler from './libs/ListHandler';
import _ from 'underscore';

export default {
  name: 'medium-editor',
  data() {
      return {
          editor: null,
          defaultOptions: {
              forcePlainText: false
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
      InsertImage,
      ListHandler
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
            
            if(this.onChange) {
                this.onChange(content)
            }
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

<style lang="css">
* {
    outline: none;
    box-sizing: border-box;
}

.medium-editor-container {
    height: 100%;
    margin: 0 auto;
    position: relative;
    padding: 2rem 0;
    font-size: 1.2rem;
}

.medium-editor-container p {
    max-width: 1000px;
    margin: 0 auto;
}


.medium-editor-container h1 {
    font-size: 2.5em;
    margin-bottom: 0.1em;
}

.medium-editor-container h2 {
    font-size: 2.2em;
    margin-top: 0.2em;
    margin-bottom: 0.1em;
}

.medium-editor-container h3 {
    font-size: 1.9em;
    margin-top: 0.2em;
    margin-bottom: 0.1em;
}

.medium-editor-container h4 {
    font-size: 1.6em;
    margin-top: 0.2em;
    margin-bottom: 0.1em;
}

.medium-editor-container h5 {
    font-size: 1.3em;
    margin-top: 0.2em;
    margin-bottom: 0.1em;
}

.medium-editor-container h6 {
    font-size: 1em;
    margin-bottom: 0.1em;
}

.medium-editor-container p {
    margin-bottom: 1em;
}
.medium-editor-container ul {
    margin-left: 30px;
    list-style: disc;
}
.medium-editor-container ul li {
    margin-bottom: 0.5em
}
.medium-editor-container ol {
    margin-left: 30px;
}
.medium-editor-container ol li {
    margin-bottom: 0.5em
}
</style>
