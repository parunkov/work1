<script setup>
import { ref } from 'vue';
import plusIcon from '../assets/plus.webp';

const props = defineProps({
    type: {
        type: String,
        required: true,
    },
    index: {
        type: Number,
        required: true,
    },
    content: {
        type: Array,
        required: true,
    }
});

const emit = defineEmits(['addLink']);
const emitAddLink = () => {
    emit('addLink', props.index);
}

const title = ref(`Card ${props.index} (type ${props.type})`)
</script>
<template>
    <div class="card">
        <div class="title">{{ title }}</div>
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