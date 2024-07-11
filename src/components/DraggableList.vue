<!-- DraggableList.vue -->
<template>
    <div class="draggable-list">
        <DraggableItem v-for="(item, index) in items" :key="item.id" :item="item" :index="index"
            @dragStart="onDragStart" @dragEnd="onDragEnd" @dragover.prevent="onDragOver(index)" @drop="onDrop(index)">
            {{ item.name }}
        </DraggableItem>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import DraggableItem from './DraggableItem.vue';

const items = ref([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' },
    { id: 4, name: 'Item 4' },
]);

const draggedIndex = ref(null);

const onDragStart = (index) => {
    draggedIndex.value = index;
};

const onDragEnd = () => {
    draggedIndex.value = null;
};

const onDragOver = (index) => {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'move';
};

const onDrop = (index) => {
    if (draggedIndex.value === null) return;
    const draggedItem = items.value[draggedIndex.value];
    items.value.splice(draggedIndex.value, 1);
    items.value.splice(index, 0, draggedItem);
    draggedIndex.value = null;
};
</script>

<style scoped>
.draggable-list {
    display: flex;
    flex-direction: column;
    gap: 10px;
}
</style>
