<template>
    <div class="image-handler-container">
        <div class="insert-image-container" v-if="insert.isShow" v-bind:style="insert.position">
            <div class="insert-image-toggle">
                <button v-on:click="toggle" class="btn-toggle">
                    <font-awesome-icon icon="plus" />
                </button>
            </div>
            <div class="insert-image-menu"  v-if="insert.isToggle">
                <file-upload
                    class="btn-toggle"
                    :post-action="uploadUrl"
                    extensions="gif,jpg,jpeg,png,webp"
                    accept="image/png,image/gif,image/jpeg,image/webp"
                    :multiple="true"
                    :size="1024 * 1024 * 10"
                    v-model="insert.files"
                    @input-filter="inputFilter"
                    @input-file="inputFile"
                    ref="upload">
                    <font-awesome-icon :icon="['far', 'images']" />
                </file-upload>
                <button class="btn-toggle" v-on:click="addEmbed">
                    <font-awesome-icon :icon="['fas', 'code']" />
                </button>
            </div>
        </div>
        <div class="image-handler" v-if="handler.isShow" v-bind:style="handler.position">
            <div class="image-hander-menu">
                <button class="btn-toggle" v-on:click="imageSizing('is-normal')">
                    <font-awesome-icon :icon="['fas', 'code']" />
                </button>
                <button class="btn-toggle" v-on:click="imageSizing('is-expand')">
                    <font-awesome-icon :icon="['fas', 'code']" />
                </button>
                <button class="btn-toggle" v-on:click="imageSizing('is-full')">
                    <font-awesome-icon :icon="['fas', 'code']" />
                </button>
            </div>
        </div>
    </div>
</template>


<script>
import VueUploadComponent from 'vue-upload-component'
import { library } from '@fortawesome/fontawesome-svg-core'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { far } from '@fortawesome/free-regular-svg-icons'
import { fas } from '@fortawesome/free-solid-svg-icons'
import EmbedJS from 'embed-js'
import github from 'embed-plugin-github'
import url from 'embed-plugin-url'
import noembed from 'embed-plugin-noembed'

library.add(fas)
library.add(far)

export default {
    components: {
        FontAwesomeIcon,
        'file-upload': VueUploadComponent
    },
    data() {
        return {
            insert: {
                position: {
                    top: '0',
                    left: '0'
                },
                isShow: false,
                isToggle: false,
                embedElm: null,
                files: [],
                focusLine: null
            },
            handler: {
                currentLine: null,
                currentImg: null,
                position: {
                    top: '0'
                },
                isShow: false
            }
        }
    },
    props: [
        'editor',
        'uploadUrl',
        'editorRef'
    ],
    methods: {
        subscribe() {
            this.editor.subscribe('editableKeyup', this.detectShowToggle)
            this.editor.subscribe('editableKeyup', this.detectEmbed)
            this.editor.subscribe('editableClick', this.detectShowToggle)
        },
        unsubscribe() {
            this.editor.unsubscribe('editableKeyup', this.detectShowToggle)
            this.editor.unsubscribe('editableKeyup', this.detectEmbed)
            this.editor.unsubscribe('editableClick', this.detectShowToggle)
        },
        detectShowToggle() {
            const currentLine = this.editor.getSelectedParentElement()
            const content = currentLine.innerHTML.replace(/^(<br\s*\/?>)+/,'').trim()
            if(content) {
                this.insert.isShow = false
                this.insert.isToggle = false
                this.insert.focusLine = null
            } else {
                const currentPos = currentLine.getBoundingClientRect();
                this.insert.position.top = currentPos.top + 'px'
                this.insert.position.left = currentPos.left + 'px'
                this.insert.isShow = true
                this.insert.focusLine = currentLine
            }
        },
        toggle () {
            this.insert.isToggle = !this.insert.isToggle;
        },
        addImage(url) {
            if(this.insert.isToggle) {
                this.editorRef.focus()
                this.editor.selectElement(this.insert.focusLine)
                this.editor.pasteHTML(`<div class="editor-image">
                    <img src="${url}" />
                    <input type="text" placeHolder="Image Description">
                </div><br />`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                this.handler.currentLine = this.editor.getSelectedParentElement().previousElementSibling
                this.handler.currentImg = this.editor.getSelectedParentElement().previousElementSibling.querySelector('img')
                this.handler.currentImg.onclick = () => {
                    this.handler.isShow = !this.handler.isShow;
                    const currentPos = this.handler.currentImg.getBoundingClientRect();
                    this.handler.position.top = currentPos.top + 'px'
                }

                this.insert.isToggle = false
                this.insert.isShow = false
                this.insert.focusLine = null
            }
        },
        imageSizing(sizing) {
            this.handler.currentLine.className = 'editor-image ' + sizing 
        },
        addEmbed() {
            if(this.insert.isToggle) {
                this.editor.pasteHTML(`<p class="editor-embed"><br></p>`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                this.insert.embedElm = this.editor.getSelectedParentElement()

                this.insert.isToggle = false
                this.insert.isShow = false
            }
        },
        detectEmbed(e) {
            if (e.keyCode === 13 && this.embedElm) {
                this.insert.embedElm.innerHTML = this.insert.embedElm.innerHTML.replace("<br>", "")
                const embed = new EmbedJS({
                    input: this.insert.embedElm,
                    plugins: [
                        url(),
                        github(),
                        noembed()
                    ]
                })
                embed.render();
                this.insert.embedElm = null
            }
        },
        inputFilter(newFile, oldFile, prevent) {
            if (newFile && !oldFile) {
                if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
                    return prevent()
                }
                if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
                    return prevent()
                }
            }
        },
        inputFile(newFile, oldFile) {
            if (newFile && !oldFile) {
                this.$refs.upload.active = true
            }
            
            // Image Upload Successful
            if(newFile && newFile.success) {
                this.addImage(newFile.response.url)
            }
        }
    },
    mounted() {
        this.subscribe()
    },
    destroyed() {
        this.unsubscribe()
    }
}
</script>

<style lang="css">
@import url('https://gist.github.com/assets/embed-0af287a4b5c981db301049e56f06e5d3.css');

* {
    outline: none;
}

.insert-image-container {
    display: flex;
    position: fixed;
    left: 100px;
    top: 100px;
    transform: translate(-54px, -7px);
}

.insert-image-container .btn-toggle {
    border: 1px solid #DDD;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 16px;
    color: #555;
}

.image-handler {
    display: flex;
    position: fixed;
    left: 50%;
    top: 100px;
    transform: translate(-50%, -20px);
    background-color: rgba(0, 0, 0, 0.6);
    border-radius: 5px;
    padding-left: 10px;
    padding-right: 10px;
}

.image-handler .btn-toggle {
    border: 0;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 16px;
    color: #FFF;
    background-color: transparent;
}
.image-handler .btn-toggle:hover {
    cursor: pointer;
    color: #00BD6A;
}

.insert-image-container .insert-image-menu {
    display: flex;
}
.insert-image-container .insert-image-menu .btn-toggle {
    margin-left: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
}
.editor-image {
    display: block;
    margin-top: 2rem;
    margin-bottom: 2rem;
}

.editor-image img {
    width: 100%;
    height: auto;
    display: block;
}

.editor-image img:hover {
    cursor: pointer;
}
.editor-image input {
    margin: 0 auto;
    border: 0;
    display: block;
    font-size: 0.7rem;
    padding: 0.5rem;
    width: 100%;
    text-align: center;
    box-sizing: border-box;
    color: #888;
    margin-bottom: 2rem;
}

.editor-embed-container.is-inactive {
    display: none;
}

.editor-embed-input.is-inactive {
    display: none;
}

.editor-embed > a {
    display: none;
}
</style>
