<template>
  <div id="app">
    <medium-editor 
        v-model="content"
        :readOnly="false"
        :prefill="defaultValue"
        :options="options"
        :onChange="onChange"
        v-on:uploaded="uploadCallback"
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
          defaultValue: `<h1>Vue2 <b>Medium Editor</b></h1>
          <div class="editor-image is-full"><img src="https://source.unsplash.com/yxNURc8he3o/2000x600"></div>
          <div class="editor-image-description">righteous indignation and dislike</div>
          <p>But I must explain to you how all this mistaken idea of denouncing <b>pleasure and praising pain was born and I will give you</b> a complete account of the system, and expound the actual teachings of the great explorer of the truth, the master-builder of human happiness. No one rejects, dislikes</p><p></p><ul><li>But I must explain to you how all this mistaken idea of denouncing</li><li>of pleasure of the moment, so blinded by desire</li></ul><p></p>
          <p>On the other hand, we denounce with righteous indignation and dislike men who are so beguiled and demoralized by the charms of pleasure of the moment, so blinded by desire, that they cannot foresee the pain and trouble that are bound to ensue; and equal blame belongs to those who fail in their duty through weakness of will</p><p class="editor-embed embed-js-applied embed-js-applied"><a href="https://gist.github.com/tui2tone/294af316b8ac09ea440c8743b97baa27" rel="" target="self">https://gist.github.com/tui2tone/294af316b8ac09ea440c8743b97baa27</a><div class="ejs-gist" data-src="https://gist.github.com/tui2tone/294af316b8ac09ea440c8743b97baa27"><iframe width="100%" id="ejs-gist-0" style="height: 285px;"></iframe></div></p><blockquote><span style="font-family: BlinkMacSystemFont, -apple-system, &quot;Segoe UI&quot;, Roboto, Oxygen, Ubuntu, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, Helvetica, Arial, sans-serif; font-size: 1.2rem; font-weight: 400;">On the other hand, we denounce with righteous indignation and dislike men who are so beguiled and demoralized by the charms of pleasure of the moment, so blinded by desire</span></blockquote><p>Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariaturasdasdasdasd<br></p>
          <h2>Section 1.10.33</h2>
          <div class="editor-image is-normal"><img src="https://source.unsplash.com/DKnXlH_r3x4/2000x800"></div>
          <div class="editor-image-description">you how all this mistaken idea of denouncing pleasure</div>
          <p>At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis praesentium voluptatum deleniti atque corrupti quos dolores et quas molestias excepturi sint occaecati cupiditate non provident, similique sunt in culpa qui officia deserunt mollitia animi, id est laborum et dolorum fuga. Et harum quidem rerum facilis est et expedita distinctio. Nam libero tempore, cum soluta nobis est eligendi optio cumque nihil impedit quo minus</p><p></p><ol><li>cum soluta nobis est eligendi optio cumque</li><li>righteous indignation and dislike</li></ol><p></p><h3>odio dignissimos ducimus</h3>
          <p></p>
          <div class="editor-image is-expand"><img src="https://source.unsplash.com/-g7axSVst6Y/1600x600">
          </div>
          <div class="editor-image-description">I will give you a complete account of the system</div>
          <p></p>
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
        console.log("change")
      },
      uploadCallback(url) {
        console.log("uploaded", url)
      }
  }
}
</script>

<style lang="css">
@import '~bulma/css/bulma.css';
@import '~medium-editor/dist/css/medium-editor.css';
@import './themes/default.css';
</style>
