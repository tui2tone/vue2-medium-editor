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
          defaultValue: `<h1>Vue2 Medium Editor</h1>
          <div class="editor-image is-full"><img src="https://source.unsplash.com/yxNURc8he3o/2000x600"></div>
          <p>But I must explain to you how all this mistaken idea of denouncing pleasure and praising pain was born and I will give you a complete account of the system, and expound the actual teachings of the great explorer of the truth, the master-builder of human happiness. No one rejects, dislikes</p>
          <p>On the other hand, we denounce with righteous indignation and dislike men who are so beguiled and demoralized by the charms of pleasure of the moment, so blinded by desire, that they cannot foresee the pain and trouble that are bound to ensue; and equal blame belongs to those who fail in their duty through weakness of will</p>
          <h2>Section 1.10.33</h2>
          <div class="editor-image"><img src="https://source.unsplash.com/DKnXlH_r3x4/2000x800"></div>
          <p>At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis praesentium voluptatum deleniti atque corrupti quos dolores et quas molestias excepturi sint occaecati cupiditate non provident, similique sunt in culpa qui officia deserunt mollitia animi, id est laborum et dolorum fuga. Et harum quidem rerum facilis est et expedita distinctio. Nam libero tempore, cum soluta nobis est eligendi optio cumque nihil impedit quo minus</p>
          <p><div class="editor-image is-expand"><img src="https://source.unsplash.com/-g7axSVst6Y/1600x600"></div></p>
          <p>On the other hand, we denounce with righteous indignation and dislike men who are so beguiled and demoralized by the charms of pleasure of the moment, so blinded by desire, that they cannot foresee the pain and trouble that are bound to ensue; and equal blame belongs to those who fail in their duty through weakness of will</p>`,
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
