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
import VueUploadComponent from 'vue-upload-component';
import { library } from '@fortawesome/fontawesome-svg-core';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { far } from '@fortawesome/free-regular-svg-icons';
import { fas } from '@fortawesome/free-solid-svg-icons';
import Gist from 'pure-gist-embed';
import randomstring from 'randomstring';

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
        'editorRef',
        'onChange'
    ],
    methods: {
        subscribe() {
            this.editor.subscribe('editableKeyup', this.detectShowToggle)
            this.editor.subscribe('editableKeyup', this.detectEmbed)
            this.editor.subscribe('editableClick', this.detectShowToggle)
            this.editor.subscribe('editableKeyup', this.detectImageDescription)
            this.editor.subscribe('editableClick', this.detectImageDescription)

            this.subscribeImageInitial()
            this.subscribeEmbedInitial()
        },
        unsubscribe() {
            this.editor.unsubscribe('editableKeyup', this.detectShowToggle)
            this.editor.unsubscribe('editableKeyup', this.detectEmbed)
            this.editor.unsubscribe('editableClick', this.detectShowToggle)
            this.editor.unsubscribe('editableKeyup', this.detectImageDescription)
            this.editor.unsubscribe('editableClick', this.detectImageDescription)
        },
        subscribeImageInitial() {
            const handlerVm = this;

            setTimeout(() => {
                const focused = this.editor.getFocusedElement()
                if(focused) {
                    const editorImages = focused.getElementsByClassName('editor-image')
                
                    for (let i = 0; i < editorImages.length; i++) {
                        editorImages[i].onclick = function() {
                            setTimeout(() => {
                                const className = this.className
                                this.className = className.replace('is-focus', '') + ' is-focus'
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
                            })
                        }
                    }
                }
            })
        },
        subscribeEmbedInitial() {
            setTimeout(() => {
                const focused = this.editor.getFocusedElement()
                if(focused) {
                    const editorEmbeds = focused.getElementsByClassName('editor-embed')

                    for (let i = 0; i < editorEmbeds.length; i++) {
                        const nextElm = editorEmbeds[i].nextElementSibling
                        const link = editorEmbeds[i].getElementsByTagName('a')[0]
                        if(link) {
                            const url = link.getAttribute('href')
                            nextElm.outerHTML = ''

                            this.renderEmbed(url, editorEmbeds[i])
                        }
                    }
                }
            })
        },
        detectImageDescription() {
            const focused = this.editor.getFocusedElement()
            if(focused) {
                const editorImages = focused.getElementsByClassName('editor-image-description')
                for (let i = 0; i < editorImages.length; i++) {
                    const descriptionElm = editorImages[i]
                    const description = descriptionElm.innerHTML.trim()

                    if(!description || description == "<br>") {
                        descriptionElm.className = 'editor-image-description is-empty'
                    } else {
                        descriptionElm.className = 'editor-image-description'
                    }
                }
            }
        },
        detectShowToggle(e) {
            if(this.insert.isShow && this.insert.isToggle) {
                this.toggle();
            }

            this.handler.isShow = false
            if(e.target.className.indexOf('editor-image-description') <= -1) {
                const editorImages = this.editor.getFocusedElement().getElementsByClassName('editor-image')
                for (let i = 0; i < editorImages.length; i++) {
                    editorImages[i].className = editorImages[i].className.replace('is-focus', '')
                }
            }

            const currentLine = this.editor.getSelectedParentElement()
            const outerHtml = currentLine.outerHTML
            const isList = outerHtml.indexOf('<li>') > -1
            const content = currentLine.innerHTML.replace(/^(<br\s*\/?>)+/,'').trim()
            if(content || isList) {
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
            this.$emit('uploaded', url);

            if(this.insert.isToggle) {
                const handlerVm = this;
                this.editorRef.focus()
                this.editor.selectElement(this.insert.focusLine)
                this.editor.pasteHTML(`<div class="editor-image">
                        <img src="${url}" />
                    </div>
                    <div class="editor-image-description"><br></div>
                    <br />`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})

                this.handler.currentLine = this.editor.getSelectedParentElement().previousElementSibling.previousElementSibling
                this.handler.currentImg = this.editor.getSelectedParentElement().previousElementSibling.previousElementSibling.querySelector('img')
                const currentPos = this.handler.currentImg.getBoundingClientRect();

                this.window.scrollTo(0, currentPos.top - currentPos.x);
                this.handler.currentLine.onclick = function() {
                    setTimeout(() => {
                        const className = this.className
                        this.className = className.replace('is-focus', '') + ' is-focus'
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
                    })
                }

                this.insert.isToggle = false
                this.insert.isShow = false
                this.insert.focusLine = null
            }
        },
        imageSizing(sizing) {
            this.handler.currentSize = sizing
            this.handler.currentLine.className = 'editor-image ' + sizing

            this.onChange()
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
            if (e.keyCode === 13 && this.insert.embedElm) {
                const url = this.insert.embedElm.innerHTML.replace("<br>", "")
                this.renderEmbed(url, this.insert.embedElm)
                this.insert.embedElm = null
            }
        },
        renderEmbed(url, elm) {
            elm.innerHTML = `
            <a href="${url}">${url}</a>
            <div class="gist-embed-iframe"></div>
            `;
            Gist.get(url)
                .then((result) => {
                    const id = randomstring.generate();
                    const iframe = document.createElement('iframe');
                    const html = result.content;
                    const iframeContainer = elm.getElementsByClassName('gist-embed-iframe')[0]
                    iframeContainer.appendChild(iframe);
                    iframe.id = id;
                    iframe.contentWindow.document.open();
                    iframe.contentWindow.document.write(html);
                    iframe.contentWindow.document.close();

                    const cssLink = document.createElement("link");
                    cssLink.href = "https://assets-cdn.github.com/assets/gist-embed-1baaff35daab552f019ad459494450f1.css"; 
                    cssLink.rel = "stylesheet"; 
                    cssLink.type = "text/css"; 
                    iframe.contentWindow.document.head.appendChild(cssLink);
                    iframe.height = iframe.contentWindow.document.body.scrollHeight + "px";
                })
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