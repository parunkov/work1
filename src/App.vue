<script setup>
import { ref, watch } from 'vue';
import Card from './components/Card.vue';
import plusIconWhite from './assets/plusWhite.svg';

// localStorage.removeItem('p1159data');
// localStorage.removeItem('p1159loadingDate');

const HOST = import.meta.env.VITE_URL_API;

const data = ref(
  localStorage.getItem('p1159data')
    ? JSON.parse(localStorage.getItem('p1159data'))
    : []
);

// console.log(data.value);
const popinOpened = ref(false);
const selectPopinOpened = ref(false);
const inputValue = ref('');
const currentIndex = ref(0);
const addInput = ref(null);
const draggedIndex = ref(null);
const isAdiingFolder = ref(false);
const isCrossPreloaderRun = ref(false);

const updateStorage = () => {
  localStorage.setItem('p1159data', JSON.stringify(data.value));
};

const getRequerst = () => {
  fetch(`${HOST}/get/`)
    .then((response) => response.json())
    .then((result) => {
      if (!result.error) {
        data.value = result;
        updateStorage();
        console.log('GET request');
      }
    });
};

const sendData = () => {
  fetch(`${HOST}/post/`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    mode: 'no-cors',
    body: JSON.stringify(data.value),
  })
    .then((response) => {
      // console.log(response);
    })
    .then(() => {
      console.log('POST request');
      document.querySelectorAll('.cross.loading').forEach((item) => {
        item.classList.remove('loading');
      });
      getRequerst();
    });
};

const loadData = () => {
  const loadingDate = localStorage.getItem('p1159loadingDate')
    ? +localStorage.getItem('p1159loadingDate')
    : 0;
  console.log('storage saved - ', new Date(loadingDate));
  console.log('update diff - ', Date.now() - loadingDate);
  if (Date.now() - loadingDate > 3600000) {
    getRequerst();
    localStorage.setItem('p1159loadingDate', Date.now());
  }
};
loadData();

const openPopin = (index) => {
  selectPopinOpened.value = false;
  popinOpened.value = true;
  currentIndex.value = index;
};

watch(popinOpened, (value) => {
  setTimeout(() => {
    if (value && addInput.value) {
      addInput.value.focus();
    }
  }, 100);
});

const onPopinButtonClick = () => {
  if (inputValue.value.length > 0) {
    if (data.value[currentIndex.value].type === 'rows') {
      data.value[currentIndex.value].content.push(inputValue.value);
    } else {
      if (isAdiingFolder.value) {
        data.value[currentIndex.value].content.push({
          folderContent: [],
          folderName: inputValue.value,
          visible: false,
        });
        isAdiingFolder.value = false;
      } else {
        data.value[currentIndex.value].content.push({
          link: inputValue.value,
          // icon: '/icons/default.png',
        });
        isAdiingFolder.value = false;
      }
    }
  }
  inputValue.value = '';
  popinOpened.value = false;
  updateStorage();
  sendData();
};

const onSelectButtonClick = (type) => {
  data.value.push({ type, content: [] });
  selectPopinOpened.value = false;
  updateStorage();
  sendData();
};

let timeoutId;
const changeCardTitle = (payload) => {
  if (payload.index === 'error') return;
  data.value[payload.index].name = payload.name;
  clearTimeout(timeoutId);
  timeoutId = setTimeout(() => {
    updateStorage();
    sendData();
  }, 500);
};

const onFolderClick = (payload) => {
  data.value[payload.index].content[payload.folderIndex].visible =
    !payload.visiblity;
  updateStorage();
  sendData();
};

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
  const draggedItem = data.value[draggedIndex.value];
  data.value.splice(draggedIndex.value, 1);
  data.value.splice(index, 0, draggedItem);
  draggedIndex.value = null;
  updateStorage();
  sendData();
};

const onOverlayClick = ({ target }) => {
  if (!target.closest('.popin')) {
    popinOpened.value = false;
    selectPopinOpened.value = false;
  }
};

let removeTimeout;
const onCrossClick = (payload) => {
  isCrossPreloaderRun.value = !isCrossPreloaderRun.value;

  clearTimeout(removeTimeout);
  removeTimeout = setTimeout(() => {
    if (
      (payload.cardType === 'tile' ||
        payload.cardType === 'row' ||
        payload.cardType === 'favorite' ||
        payload.cardType === 'folder') &&
      isCrossPreloaderRun.value
    ) {
      data.value[payload.cardIndex].content.splice(payload.itemIndex, 1);
      isCrossPreloaderRun.value = false;
      updateStorage();
      sendData();
    }

    if (payload.cardType === 'favoriteFolder') {
      console.log(
        data.value[payload.cardIndex].content[payload.folderIndex]
          ?.folderContent
      );
    }
  }, 2000);

  if (!isCrossPreloaderRun.value) {
    document.querySelectorAll('.cross.loading').forEach((item) => {
      item.classList.remove('loading');
    });
    clearTimeout(removeTimeout);
  }
};

const onAddFolder = (index) => {
  openPopin(index);
  isAdiingFolder.value = true;
};

const onItemDrop = (payload) => {
  const draggedItem = data.value[payload.cardIndex].content[payload.draggingIndex];
  data.value[payload.cardIndex].content.splice(payload.draggingIndex, 1);
  data.value[payload.cardIndex].content.splice(payload.index, 0, draggedItem);
  draggedIndex.value = null;
  updateStorage();
  sendData();
}
</script>

<template>
  <div class="wrapper">
    <div class="container">
      <Card
        v-for="(item, index) in data"
        :key="index"
        :type="item.type"
        :index="index"
        :name="item.name"
        :content="item.content"
        @addLink="openPopin(index)"
        @cangeTitle="changeCardTitle"
        @folderClick="onFolderClick"
        @dragStart="onDragStart"
        @dragEnd="onDragEnd"
        @dragover.prevent="onDragOver(index)"
        @drop="onDrop(index)"
        @crossClick="onCrossClick"
        @addFolderLink="onAddFolder(index)"
        @itemDrop="onItemDrop"
      />
      <button
        class="selectButton"
        @click="
          selectPopinOpened = true;
          popinOpened = false;
        "
        style="--span: 3"
      >
        <img :src="plusIconWhite" alt="" />
      </button>
    </div>
    <div v-if="popinOpened" class="overlay" @click="onOverlayClick">
      <div class="popin">
        <input
          type="text"
          v-model="inputValue"
          class="popinInput"
          ref="addInput"
          @keyup.enter="onPopinButtonClick"
        />
        <button class="popinButton" @click="onPopinButtonClick">Add</button>
      </div>
    </div>
    <div v-if="selectPopinOpened" class="overlay" @click="onOverlayClick">
      <div class="popin">
        <button
          class="popinButton select"
          @click="onSelectButtonClick('tiles')"
        >
          Tile
        </button>
        <button class="popinButton select" @click="onSelectButtonClick('rows')">
          Row
        </button>
        <button
          class="popinButton select"
          @click="onSelectButtonClick('favorites')"
        >
          Faworite
        </button>
      </div>
    </div>
  </div>
</template>

<style>
.wrapper {
  position: relative;

  * {
    font-size: 0.83vw;
    line-height: 1.04vw;

    @media screen and (max-width: 1023px) {
      font-size: 3.2vw;
      line-height: 4vw;
    }
  }
}

.container {
  padding-top: 1.4vw;
  padding-left: 1vw;
  padding-right: 1vw;
  margin-bottom: 1.4vw;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1vw;
  grid-auto-rows: 1px;
  grid-auto-flow: dense;

  @media screen and (max-width: 1023px) {
    display: block;
  }
}

.card {
  padding-top: 0.56vw;
  padding-left: 0.84vw;
  padding-right: 0.84vw;
  border-radius: 1.4vw;
  background: #282828;
  grid-row-end: span var(--span, 1);
  max-width: 32vw;

  @media screen and (max-width: 1023px) {
    padding-bottom: 0.56vw;
    margin-bottom: 1vw;
    max-width: 98vw;
  }
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
}

.popin {
  position: fixed;
  top: 50%;
  left: 50%;
  padding: 1.4vw;
  transform: translate(-50%, -50%);
  border-radius: 1.4vw;
  background: #282828;
}

.popinInput {
  width: 34.7vw;
  border-radius: 0.7vw;
  background: #d9d9d9;
  padding: 0.24vw 0.7vw;
  margin-bottom: 0.34vw;
  color: #282828;
  border: none;
  outline: none;
}

.popinButton {
  display: block;
  margin-top: 0.7vw;
  border-radius: 0.7vw;
  background: #d9d9d9;
  padding: 0.24vw 0.7vw;
  margin-bottom: 0.34vw;
  color: #282828;
  border: none;
  outline: none;

  &.select {
    text-align: center;
    width: 100%;
    min-width: 8.4vw;
    cursor: pointer;
  }
}

.selectButton {
  display: block;
  cursor: pointer;
  border-radius: 1.4vw;
  background: #282828;
  border: none;
  outline: none;
  grid-row-end: span var(--span, 1);

  @media screen and (max-width: 1023px) {
    width: 100%;
  }

  img {
    display: block;
    width: 1.4vw;
    margin: auto;

    @media screen and (max-width: 1023px) {
      width: 4vw;
    }
  }
}
</style>
