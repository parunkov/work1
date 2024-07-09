<script setup>
import { ref, watch } from 'vue';
import plusIcon from '../assets/plus.svg';

const props = defineProps({
    type: { type: String, required: true, },
    title: { type: String, required: true },
    index: { type: Number, required: true, },
    content: { type: Array, required: true, }
});

const emit = defineEmits(['addLink', 'cangeTitle']);

const setSpanHeight = (content) => {
    const columnsValue = props.type === 'tile' ? 8 : 1;
    const rowHeight = props.type === 'tile' ? 3.6 : 2.06;
    const rowsValue = Math.ceil((content.length + 1) / columnsValue);
    return Math.ceil(rowsValue * rowHeight + 3);
};

const spanHeight = ref(setSpanHeight(props.content));
watch(props.content, (content) => {
    spanHeight.value = setSpanHeight(content);
})

const emitAddLink = () => {
    emit('addLink', props.index);
}

emit('cangeTitle', { index: props.index, title: `Card ${props.index + 1} (type ${props.type})` });

const inputTitle = (event) => {
    const payload = { index: props.index, title: event.target.value };
    emit('cangeTitle', payload);
}
</script>
<template>
    <div class="card" :style="`--span: ${spanHeight};`">
        <input type="text" class="titleInput" :value="title" @input="inputTitle">
        <div :class="['cardContent', props.type === 'tile' ? 'cardContent_type_tile' : 'cardContent_type_row']">
            <div v-for="item in props.content" :key="item.link" :class="props.type">
                <img v-if="props.type === 'tile'" :src="item.icon" alt="">
                <div v-else>
                    {{ item.link }}
                </div>
            </div>
            <div :class="['plus', props.type === 'tile' ? 'tile' : 'row']" @click="emitAddLink">
                <img :src="plusIcon" alt="">
            </div>
        </div>
    </div>
</template>
<style scoped>
.titleInput {
    display: block;
    width: 100%;
    margin-bottom: 0.7vw;
    border-radius: 0.7vw;
    border: none;
    outline: none;
    background: #282828;
    color: #ffffff;
}
</style>