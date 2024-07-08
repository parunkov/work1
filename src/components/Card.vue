<script setup>
import { ref } from 'vue';
import plusIcon from '../assets/plus.svg';

const props = defineProps({
    type: { type: String, required: true, },
    title: { type: String, required: true },
    index: { type: Number, required: true, },
    content: { type: Array, required: true, }
});

const emit = defineEmits(['addLink', 'cangeTitle']);
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
    <div class="card">
        <!-- <div class="title">{{ title }}</div> -->
        <input type="text" class="titleInput" :value="title" @input="inputTitle">
        <div class="cardContent">
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
    font-size: 12px;
    line-height: 15px;
    margin-bottom: 10px;
    border-radius: 10px;
    border: none;
    outline: none;
    background: #282828;
    color: #ffffff;
}
</style>