<template>
    <div class="list-handler">
    </div>
</template>


<script>

export default {
    components: {
    },
    data() {
        return {
            focusLine: null,
            isLastBeforeLi: false
        }
    },
    props: [
        'editor'
    ],
    methods: {
        subscribe() {
            this.editor.subscribe('editableKeydown', this.detectList)
        },
        unsubscribe() {
            this.editor.unsubscribe('editableKeydown', this.detectList)
        },
        detectList(e) {
            const currentLine = this.editor.getSelectedParentElement()
            const matchList = currentLine.innerHTML.match(/^(- )/g)
            
            if(matchList && matchList.length > 0) {
                const content = currentLine.innerHTML.replace(/^(- )/g, "")
                currentLine.innerHTML = "<ul><li>" + content + "</li></ul>"
            }

            const matchOlList = currentLine.innerHTML.match(/^(\d+. )/g)
            
            if(matchOlList && matchOlList.length > 0) {
                const content = currentLine.innerHTML.replace(/^(\d+. )/g, "")
                currentLine.innerHTML = "<ol><li>" + content + "</li></ol>"
            }

            const key = e.keyCode || e.charCode;
            if( key == 8 || key == 46 ) {

                if(currentLine.outerHTML == '<li><br></li>') {
                    this.isLastBeforeLi = true
                } else {
                    this.isLastBeforeLi = false
                }

                if(this.isLastBeforeLi) {
                    currentLine.outerHTML = ""
                    this.editor.pasteHTML(`<p><br><br></p>`, { cleanAttrs: [], cleanTags: [], unwrapTags: []})
                }
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
</style>
