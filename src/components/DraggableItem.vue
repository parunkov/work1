<!-- DraggableItem.vue -->
<template>
    <div class="draggable-item" :draggable="true" @dragstart="onDragStart" @dragend="onDragEnd">
        <slot></slot>
    </div>
</template>

<script setup>
import { defineProps, defineEmits, ref } from 'vue';

const props = defineProps({
    item: Object,
    index: Number,
});

const emit = defineEmits(['dragStart', 'dragEnd']);
const dragging = ref(false);

const onDragStart = (event) => {
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('text/plain', props.index);
    dragging.value = true;
    emit('dragStart', props.index);
};

const onDragEnd = () => {
    dragging.value = false;
    emit('dragEnd');
};
</script>

<style scoped>
.draggable-item {
    cursor: grab;
}

.draggable-item.dragging {
    opacity: 0.5;
}
</style>
