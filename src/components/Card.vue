<script setup>
import { onMounted, onUpdated, ref, watch } from 'vue';
import plusIcon from '../assets/plus.svg';
import rightIcon from '../assets/triangle-right.svg';
import downIcon from '../assets/triangle-down.svg';
import iconPatch from '../assets/icon.png';
import { crossMarkup } from '../assets/icons';

const HOST = import.meta.env.VITE_URL_API;

const props = defineProps({
  type: { type: String, required: true },
  name: { type: String },
  index: { type: Number, required: true },
  content: { type: Array, required: true },
});

const emit = defineEmits([
  'addLink',
  'cangeTitle',
  'folderClick',
  'dragStart',
  'dragEnd',
  'crossClick',
  'addFolderLink',
]);

const dragging = ref(false);

const setSpanHeight = (content) => {
  const columnsValue = props.type === 'tiles' ? 8 : 1;
  const rowHeight = props.type === 'tiles' ? 3.6 : 2.06;
  const contentValue = props.content?.reduce(
    (summ, item) =>
      summ +
      (props.type === 'favorites' && item.folderContent && item.visible
        ? item.folderContent.length + 1
        : 1),
    1
  );
  const rowsValue = Math.ceil(contentValue / columnsValue);
  return Math.ceil(
    rowsValue * rowHeight + (props.type === 'favorites' ? 5 : 3)
  );
};

const spanHeight = ref(setSpanHeight(props.content));
watch(props.content, (content) => {
  spanHeight.value = setSpanHeight(content);
});

onMounted(() => {
  spanHeight.value = setSpanHeight(props.content);
});

onUpdated(() => {
  spanHeight.value = setSpanHeight(props.content);
});

const emitAddLink = () => {
  emit('addLink', props.index);
  setTimeout(() => {
    spanHeight.value = setSpanHeight(props.content);
  }, 100);
};

const emitAddFolderLink = () => {
  emit('addFolderLink', props.index);
  setTimeout(() => {
    spanHeight.value = setSpanHeight(props.content);
  }, 100);
};

if (!props.name)
  emit('cangeTitle', {
    index: props.index,
    name: `Card ${props.index + 1} (type ${props.type})`,
  });

const inputTitle = (event) => {
  const payload = { index: props.index, name: event.target.value };
  emit('cangeTitle', payload);
};

const onFolderClick = (event) => {
  const folderIndex =
    event.target.closest('.favoriteFolder')?.dataset?.folderIndex;
  const visiblity = props.content[folderIndex].visible;
  const payload = { index: props.index, folderIndex, visiblity };
  emit('folderClick', payload);
  setTimeout(() => {
    spanHeight.value = setSpanHeight(props.content);
  }, 100);
};

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

const onCrossClick = (event) => {
  event.target.closest('.cross')?.classList.add('loading');

  const card = event.target.closest('.card');
  const cards = [...document.querySelectorAll('.card')];
  const cardIndex = cards.findIndex((element) => element === card);

  let itemIndex;
  let cardType;
  let folderIndex;

  const tiles = [...card.querySelectorAll('.tile')];
  const tile = event.target.closest('.tile');

  if (tile) {
    itemIndex = tiles.findIndex((element) => element === tile);
    cardType = 'tile';
  }

  const rows = [...card.querySelectorAll('.row')];
  const row = event.target.closest('.row');

  if (row) {
    itemIndex = rows.findIndex((element) => element === row);
    cardType = 'row';
  }

  const folder = event.target.closest('.favoriteFolder');
  const favorite = event.target.closest('.favorite');
  const wrapper = event.target.closest('.favoriteWrapper');
  const wrappers = [...card.querySelectorAll('.favoriteWrapper')];

  if (folder) {
    folderIndex = wrappers.findIndex((element) => element === wrapper);
    const folderItem = event.target.closest('.favorite_folder_item');
    const folderItems = [...folder.querySelectorAll('.favorite_folder_item')];
    itemIndex = folderItems.findIndex((element) => element === folderItem);
    cardType = 'favoriteFolder';
  } else if (favorite) {
    itemIndex = wrappers.findIndex((element) => element === wrapper);
    cardType = 'favorite';
  }

  const payload = { cardIndex, cardType, itemIndex, folderIndex };
  emit('crossClick', payload);
};
</script>

<template>
  <div class="card" :style="`--span: ${spanHeight};`" :draggable="true" @dragstart="onDragStart" @dragend="onDragEnd">
    <input type="text" class="titleInput" :value="name" @input="inputTitle" />

    <div v-if="props.type === 'tiles'" class="cardContent cardContent_type_tile">
      <div v-for="item in props.content" :key="item.link" class="tile">
        <img :src="item.icon ? `${HOST}${item.icon}` : iconPatch" />
        <div class="cross cross_type_tile" @click="onCrossClick" v-html="crossMarkup"></div>
      </div>
      <div class="plus tile" @click="emitAddLink">
        <img :src="plusIcon" alt="" />
      </div>
    </div>

    <div v-if="props.type === 'rows'" class="cardContent cardContent_type_row">
      <div v-for="item in props.content" :key="item.link" class="row">
        <div class="rowContent">
          {{ item }}
        </div>
        <div class="cross cross_type_row" @click="onCrossClick" v-html="crossMarkup"></div>
      </div>
      <div class="plus row" @click="emitAddLink">
        <img :src="plusIcon" alt="" />
      </div>
    </div>

    <div v-if="props.type === 'favorites'" class="cardContent cardContent_type_row">
      <div v-for="(item, folderIndex) in props.content" :key="item.link" class="favoriteWrapper">
        <div v-if="item.folderName" class="favoriteFolder" :data-folder-index="folderIndex">
          <div v-if="item.folderName" class="favorite favoriteFolderTitle" @click="onFolderClick">
            <img :src="item.visible ? downIcon : rightIcon" />
            {{ item.folderName }}
          </div>
          <div v-if="item.visible" v-for="itemLink in item.folderContent" class="favorite favorite_folder_item">
            <img :src="itemLink.icon ? `${HOST}${itemLink.icon}` : iconPatch" />
            {{ itemLink.link }}
            <div class="cross cross_type_favorite" @click="onCrossClick" v-html="crossMarkup"></div>
          </div>
        </div>
        <div v-else class="favorite">
          <img :src="item.icon ? `${HOST}${item.icon}` : iconPatch" />
          <span class="favoriteLink">{{ item.link }}</span>
          <div class="cross cross_type_favorite" @click="onCrossClick" v-html="crossMarkup"></div>
        </div>
      </div>
      <div class="plus plus_type_folder row" @click="emitAddFolderLink">
        <img class="plusFolderIcon" :src="plusIcon" alt="" />
        <div class="plusFolderText">folder</div>
      </div>
      <div class="plus row" @click="emitAddLink">
        <img :src="plusIcon" alt="" />
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
  position: relative;
  border-radius: 0.7vw;
  background: #d9d9d9;
  padding: 0.7vw;

  img {
    display: block;
    width: 100%;
    margin: auto;
  }
}

.cross {
  position: absolute;
  width: 0.87vw;
  height: 0.87vw;
  border-radius: 0.435vw;
  background: #3a3a3a;
  cursor: pointer;
  @media screen and (max-width: 1023px) {
    width: 2vw;
    height: 2vw;
    border-radius: 1vw;
  }
}

.cross_type_tile {
  top: 0.087vw;
  right: 0.087vw;
}

.cross_type_row,
.cross_type_favorite {
  top: 50%;
  right: 0.087vw;
  transform: translateY(-50%);
}

.title {
  margin-bottom: 0.7vw;
}

.row {
  position: relative;
  width: 100%;
  border-radius: 0.7vw;
  background: #d9d9d9;
  padding: 0.24vw 0.7vw;
  padding-right: 1.57vw;
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

.rowContent {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.favorite {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
  border-radius: 0.7vw;
  background: #282828;
  padding: 0.24vw 0.7vw;
  padding-right: 1.57vw;
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

.favoriteLink {
  display: block;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.favorite_folder_item {
  padding-left: 30px;
}

.favoriteFolderTitle {
  cursor: pointer;
}

.plusFolderIcon {
  width: 1.1vw;
  margin: 0 !important;
}

.plusFolderText {
  margin-left: 0.42vw;
}

.plus_type_folder {
  display: flex;
  justify-content: center;
}
</style>

<style>
.crossIcon,
.crossIcon svg,
.crossLoader,
.crossLoader svg {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 0.7vw;
  height: 0.7vw;
  @media screen and (max-width: 1023px) {
    width: 1.6vw;
    height: 1.6vw;
  }
}
.cross:hover .crossIcon svg {
  fill: #FFA500;
}

.crossLoader,
.crossLoader svg {
  display: none;
}

.cross.loading {
  cursor: default;

  .crossIcon,
  .crossIcon svg {
    display: none;
  }

  .crossLoader,
  .crossLoader svg {
    display: block;
  }
}
</style>
