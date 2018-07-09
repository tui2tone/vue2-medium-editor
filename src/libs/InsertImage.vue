<template>
    <div class="insert-image-container" v-if="isShow" v-bind:style="position">
        <div class="insert-image-toggle">
            <button v-on:click="toggle" class="btn-toggle">
                <font-awesome-icon icon="plus" />
            </button>
        </div>
        <div class="insert-image-menu" v-if="isToggle">
            <button class="btn-toggle">
                <font-awesome-icon :icon="['far', 'images']" />
            </button>
        </div>
    </div>
</template>


<script>
import { library } from '@fortawesome/fontawesome-svg-core'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { far } from '@fortawesome/free-regular-svg-icons'
import { fas } from '@fortawesome/free-solid-svg-icons'

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
            isToggle: false
        }
    },
    props: [
        'editor'
    ],
    methods: {
        subscribe() {
            this.editor.subscribe('editableKeyup', this.detectShowToggle)
            this.editor.subscribe('editableClick', this.detectShowToggle)
        },
        unsubscribe() {
            this.editor.unsubscribe('editableKeyup', this.detectShowToggle)
            this.editor.unsubscribe('editableClick', this.detectShowToggle)
        },
        detectShowToggle() {
            const currentLine = this.editor.getSelectedParentElement()
            const content = currentLine.innerHTML.replace(/^(<br\s*\/?>)+/,'').trim()
            if(content) {
                this.isShow = false
            } else {
                const currentPos = currentLine.getBoundingClientRect();
                this.position.top = currentPos.top + 'px'
                this.position.left = currentPos.left + 'px'
                this.isShow = true
            }
        },
        toggle () {
            this.isToggle = !this.isToggle;
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

<style lang="css" scoped>

* {
    outline: none;
}
.insert-image-container {
    display: flex;
    position: fixed;
    transform: translate(-50px, -8px);
}
.insert-image-toggle {

}
.btn-toggle {
    border: 1px solid #DDD;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    font-size: 16px;
    color: #555;
}

.insert-image-menu {
}
.insert-image-menu .btn-toggle {
    margin-left: 10px;
}
</style>
