<script setup lang="ts">
    import { MdEditor } from 'md-editor-v3'
    import 'md-editor-v3/lib/style.css'
    const text = ref()
    const props = defineProps({
        content: {
            required: false,
            type: String,
            default: '',
        },
    })
    watch(
        () => props.content,
        () => {
            text.value = props.content
        }
    )

    const emit = defineEmits(['sendText'])

    function showHTML() {
        // emit('sendText', text.value);
        setTimeout(() => {
            emit('sendText', text.value)
        }, 1)
    }
</script>
<template class="z-[10]">
    <MdEditor
        v-model="text"
        :preview="false"
        :no-upload-img="true"
        language="en-US"
        @on-change="showHTML" />
</template>
