<template>
  <div id="app">
    <medium-editor 
        v-model="content"
        :prefill="defaultValue"
        :options="options"
        :onChange="onChange"
        ></medium-editor>
  </div>
</template>

<script>
import Editor from './Editor.vue'

export default {
  name: 'app',
  data() {
      return {
          content: ``,
          defaultValue: `<p>Test</p>`,
          options: {
            uploadUrl: "http://localhost:3000/v1/upload/image"
          }
      }
  },
  components: {
    'medium-editor': Editor
  },
  methods: {
      onChange() {
        // eslint-disable-next-line no-console
        console.log("change", this.content)
      }
  }
}
</script>

<style lang="css">
@import '~bulma/css/bulma.css';
@import '~medium-editor/dist/css/medium-editor.css';
@import './themes/default.css';
</style>
