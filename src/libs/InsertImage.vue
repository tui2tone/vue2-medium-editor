<template>
    <div>Insert Image {{a}}</div>
</template>


<script>
export default {
    data() {
        return {
            a: 2
        }
    },
    props: [
        'editor'
    ],
    methods: {
        handleKeydown() {
            this.a++;
        },
        subscribe() {
            this.editor.subscribe('editableInput', () => {
                // eslint-disable-next-line no-console
                console.log("added")
            })
        }
    },
    mounted() {
        this.subscribe()
    }
}
</script>
