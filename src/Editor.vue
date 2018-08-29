<template>
    <div>
        <div class="medium-editor-container" v-if="!readOnly">
            <insert-image v-if="editor" :uploadUrl="options.uploadUrl" :onChange="triggerChange" v-on:uploaded="uploadedCallback" :editorRef="$refs.editor" :editor="editor"></insert-image>
            <list-handler v-if="editor" :editor="editor" :onChange="triggerChange"></list-handler>
            <div class="editor" v-bind:class="{'has-content': hasContent}" v-html="prefill" ref="editor">
            </div>
        </div>
        <div class="medium-editor-container" v-if="readOnly">
            <div class="editor read-only has-content" v-html="prefill">
            </div>
        </div>
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
            forcePlainText: false,
            placeholder: {
                text: 'Write something great!!'
            },
            toolbar: {
                buttons: ['bold', 'italic', 'quote', 'h1', 'h2', 'h3', 'h4', 'h5'],
            }
          },
          hasContent: false
      }
  },
  props: [
      'options',
      'onChange',
      'prefill',
      'readOnly'
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
      if(!this.readOnly) {
        this.createElm();
      }
  },
  methods: {
      createElm() {
        this.editor = new MediumEditor(this.$refs.editor, this.editorOptions)

        if(this.prefill) {
            if(/<[a-z][\s\S]*>/i.test(this.prefill)) {
                this.hasContent = true
            } else {
                this.hasContent = false
            }
            this.$refs.editor.focus()
        }

        this.editor.subscribe('editableInput', this.triggerChange)
      },
      destroyElm() {
        this.editor.destroy()
      },
      triggerChange() {
        const content = this.editor.getContent()

        setTimeout(() => {
            if(/<[a-z][\s\S]*>/i.test(content)) {
                this.hasContent = true
            } else {
                this.hasContent = false
            }
        }, 0)
        
        this.$emit('input', content)
        
        if(this.onChange) {
            this.onChange(content)
        }
      },
      uploadedCallback(url) {
          this.$emit('uploaded', url)
      }
  },
  destroyed() {
      this.destroyElm()
  }
}
</script>
