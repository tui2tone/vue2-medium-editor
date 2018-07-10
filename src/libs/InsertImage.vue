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
                    <img src="../assets/icons/image-align-normal-active.png" v-if="handler.currentSize == 'is-normal'">
                    <img src="../assets/icons/image-align-normal.png" v-if="handler.currentSize != 'is-normal'">
                </button>
                <button class="btn-toggle" v-on:click="imageSizing('is-expand')">
                    <img src="../assets/icons/image-align-expand-active.png" v-if="handler.currentSize == 'is-expand'">
                    <img src="../assets/icons/image-align-expand.png" v-if="handler.currentSize != 'is-expand'">
                </button>
                <button class="btn-toggle" v-on:click="imageSizing('is-full')">
                    <img src="../assets/icons/image-align-full-active.png" v-if="handler.currentSize == 'is-full'">
                    <img src="../assets/icons/image-align-full.png" v-if="handler.currentSize != 'is-full'">
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
                currentSize: 'is-normal',
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

            this.subscribeImageInitial()
        },
        unsubscribe() {
            this.editor.unsubscribe('editableKeyup', this.detectShowToggle)
            this.editor.unsubscribe('editableKeyup', this.detectEmbed)
            this.editor.unsubscribe('editableClick', this.detectShowToggle)
        },
        subscribeImageInitial() {
            const handlerVm = this;

            setTimeout(() => {
                const editorImages = this.editor.getFocusedElement().getElementsByClassName('editor-image')
            
                for (let i = 0; i < editorImages.length; i++) {
                    editorImages[i].onclick = function() {
                        const className = this.className
                        if(className.indexOf('expand') > -1) {
                            handlerVm.handler.currentSize = 'is-expand'
                        } else if(className.indexOf('full') > -1) {
                            handlerVm.handler.currentSize = 'is-full'
                        } else {
                            handlerVm.handler.currentSize = 'is-normal'
                        }
                        const img = this.querySelector('img')
                        handlerVm.handler.currentLine = this;
                        handlerVm.handler.isShow = !handlerVm.handler.isShow;
                        const currentPos = img.getBoundingClientRect();
                        handlerVm.handler.position.top = currentPos.top + 'px'
                    }
                }
            })
        },
        detectShowToggle() {
            if(this.insert.isShow && this.insert.isToggle) {
                this.toggle();
            }

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
                const handlerVm = this;
                this.editorRef.focus()
                this.editor.selectElement(this.insert.focusLine)
                this.editor.pasteHTML(`<div class="editor-image">
                    <img src="${url}" />
                    <input type="text" placeHolder="Image Description">
                </div><br />`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                this.handler.currentLine = this.editor.getSelectedParentElement().previousElementSibling
                this.handler.currentImg = this.editor.getSelectedParentElement().previousElementSibling.querySelector('img')
                const currentPos = this.handler.currentImg.getBoundingClientRect();

                this.window.scrollTo(0, currentPos.top);
                this.handler.currentLine.onclick = function() {
                        const className = this.className
                    if(className.indexOf('expand') > -1) {
                        handlerVm.handler.currentSize = 'is-expand'
                    } else if(className.indexOf('full') > -1) {
                        handlerVm.handler.currentSize = 'is-full'
                    } else {
                        handlerVm.handler.currentSize = 'is-normal'
                    }

                    const img = this.querySelector('img')
                    handlerVm.handler.currentLine = this;
                    handlerVm.handler.isShow = !handlerVm.handler.isShow;
                    const currentPos = img.getBoundingClientRect();
                    handlerVm.handler.position.top = currentPos.top + 'px'
                }

                this.insert.isToggle = false
                this.insert.isShow = false
                this.insert.focusLine = null
            }
        },
        imageSizing(sizing) {
            this.handler.currentSize = sizing
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
        },
        handleScroll() {
            this.handler.isShow = false

            if(this.insert.isShow) {
                const currentLine = this.editor.getSelectedParentElement()
                const currentPos = currentLine.getBoundingClientRect();
                this.insert.position.top = currentPos.top + 'px'
                this.insert.position.left = currentPos.left + 'px'
                this.insert.isShow = true
                this.insert.focusLine = currentLine
            }
        }
    },
    mounted() {
        this.subscribe()
    },
    destroyed() {
        this.unsubscribe()
    },
    beforeMount () {
        this.window = window;
        window.addEventListener('scroll', this.handleScroll);
    },
    beforeDestroy () {
        window.removeEventListener('scroll', this.handleScroll);
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
    background-color: #FFF;
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
    max-width: 1000px;
    margin: 2rem auto;
}

.editor-image img {
    width: 100%;
    height: auto;
    display: block;
}

.editor-image img:hover {
    cursor: pointer;
}
.editor-image.is-expand {
    max-width: 1200px;
}
.editor-image.is-full {
    max-width: 100%;
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
