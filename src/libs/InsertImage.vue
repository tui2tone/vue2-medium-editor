<template>
    <div class="insert-image-container" v-if="isShow" v-bind:style="position">
        <div class="insert-image-toggle">
            <button v-on:click="toggle" class="btn-toggle">
                <font-awesome-icon icon="plus" />
            </button>
        </div>
        <div class="insert-image-menu"  v-if="isToggle">
            <button class="btn-toggle" v-on:click="addImage">
                <font-awesome-icon :icon="['far', 'images']" />
            </button>
            <button class="btn-toggle" v-on:click="addEmbed">
                <font-awesome-icon :icon="['fas', 'code']" />
            </button>
        </div>
    </div>
</template>


<script>
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
        FontAwesomeIcon
    },
    data() {
        return {
            isShow: false,
            position: {
                top: '0',
                left: '0'
            },
            isToggle: false,
            embedElm: null
        }
    },
    props: [
        'editor'
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
                this.isShow = false
                this.isToggle = false
            } else {
                const currentPos = currentLine.getBoundingClientRect();
                this.position.top = currentPos.top + 'px'
                this.position.left = currentPos.left + 'px'
                this.isShow = true
            }
        },
        toggle () {
            this.isToggle = !this.isToggle;
        },
        addImage() {
            if(this.isToggle) {
                this.editor.pasteHTML(`<div class="editor-image">
                    <img src="http://via.placeholder.com/350x150" />
                    <input type="text" placeHolder="Image Description">
                </div><br />`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                // const img = this.editor.getSelectedParentElement().querySelector('img')
                // img.className = 'editor-image'
                // const input = this.editor.getSelectedParentElement().querySelector('input')
                // input.className = 'editor-image-input'
                // img.onclick = () => {
                    
                // }

                // input.onclick = () => {
                    
                // }

                // input.onkeypress = (e) => {
                //     if (e.keyCode === 13) {
                //         this.editor.getFocusedElement().focus()
                //     }
                // }
                this.isToggle = false
                this.isShow = false
            }
        },
        addEmbed() {
            if(this.isToggle) {
                this.editor.pasteHTML(`<p class="editor-embed"><br></p>`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                this.embedElm = this.editor.getSelectedParentElement()

                this.isToggle = false
                this.isShow = false
            }
        },
        detectEmbed(e) {
            if (e.keyCode === 13 && this.embedElm) {
                this.embedElm.innerHTML = this.embedElm.innerHTML.replace("<br>", "")
                // eslint-disable-next-line no-console
                console.log(this.embedElm)
                const embed = new EmbedJS({
                    input: this.embedElm,
                    plugins: [
                        url(),
                        github(),
                        noembed()
                    ]
                })
                embed.render();
                this.embedElm = null
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
    transform: translate(-50px, -8px);
}
.insert-image-container .insert-image-toggle {

}
.insert-image-container .btn-toggle {
    border: 1px solid #DDD;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 16px;
    color: #555;
}

.insert-image-container .insert-image-menu {
}
.insert-image-container .insert-image-menu .btn-toggle {
    margin-left: 10px;
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
