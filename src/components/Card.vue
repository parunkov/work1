<script setup>
import { ref, watch } from 'vue';
import plusIcon from '../assets/plus.svg';
import rightIcon from '../assets/triangle-right.svg';
import downIcon from '../assets/triangle-down.svg';

const icons = import.meta.glob('../assets/icons/*.jpg', { eager: true });

const props = defineProps({
    type: { type: String, required: true, },
    title: { type: String, required: true },
    index: { type: Number, required: true, },
    content: { type: Array, required: true, }
});

const emit = defineEmits(['addLink', 'cangeTitle']);

const setSpanHeight = (content) => {
    const columnsValue = props.type === 'tiles' ? 8 : 1;
    const rowHeight = props.type === 'tiles' ? 3.6 : 2.06;
    const contentValue = props.content.reduce((summ, item) => summ + (props.type === 'favorites' && item.folderContent && item.visible ? item.folderContent.length + 1 : 1), 1);
    console.log(contentValue);
    const rowsValue = Math.ceil(contentValue / columnsValue);
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
        <div v-if="props.type === 'tiles'" class="cardContent cardContent_type_tile">
            <div v-for="item in props.content" :key="item.link" class="tile">
                <img :src="icons[`../assets${item.icon}`].default">
            </div>
            <div class="plus tile" @click="emitAddLink">
                <img :src="plusIcon" alt="">
            </div>
        </div>
        <div v-if="props.type === 'rows'" class="cardContent cardContent_type_row">
            <div v-for="item in props.content" :key="item.link" class="row">
                <div>
                    {{ item }}
                </div>
            </div>
            <div class="plus row" @click="emitAddLink">
                <img :src="plusIcon" alt="">
            </div>
        </div>
        <div v-if="props.type === 'favorites'" class="cardContent cardContent_type_row">
            <div v-for="item in props.content" :key="item.link">
                <div v-if="item.folderName" class="favoriteFolder">
                    <div v-if="item.folderName" class="favorite favoriteFolderTitle"
                        @click="item.visible = !item.visible">
                        <img :src="item.visible ? downIcon : rightIcon">
                        {{ item.folderName }}
                    </div>
                    <div v-if="item.visible" v-for="itemLink in item.folderContent"
                        class="favorite favorite_folder_item">
                        <img :src="icons[`../assets${itemLink.icon}`].default">
                        {{ itemLink.link }}
                    </div>
                </div>
                <div v-else class="favorite">
                    <img :src="icons[`../assets${item.icon}`].default">
                    {{ item.link }}
                </div>
            </div>
            <div class="plus row" @click="emitAddLink">
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

.cardContent_type_tile {
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    gap: 0.35vw;

    @media screen and (max-width: 1023px) {
        gap: 1vw;
    }
}

.tile {
    border-radius: 0.7vw;
    background: #D9D9D9;
    padding: 0.7vw;

    img {
        display: block;
        width: 100%;
        margin: auto;
    }
}

.title {
    margin-bottom: 0.7vw;
}

.row {
    width: 100%;
    border-radius: 0.7vw;
    background: #D9D9D9;
    padding: 0.24vw 0.7vw;
    margin-bottom: 0.69vw;
    color: #282828;

    img {
        display: block;
        width: 1.1vw;
        margin: auto;

        @media screen and (max-width: 1023px) {
            width: 4vw;
        }
    }
}

.favorite {
    width: 100%;
    border-radius: 0.7vw;
    background: #282828;
    padding: 0.24vw 0.7vw;
    margin-bottom: 0.69vw;
    color: #ffffff;

    img {
        width: 0.8vw;
        height: 0.8vw;
        margin-right: 5px;

        @media screen and (max-width: 1023px) {
            height: 4vw;
            width: 4vw;
        }
    }
}

.favorite_folder_item {
    padding-left: 30px;
}

.favoriteFolderTitle {
    cursor: pointer;
}
</style>