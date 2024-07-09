<script setup>
import { ref, watch } from 'vue';
import iconPatch from './assets/icon.png';
import Card from './components/Card.vue';
import plusIconWhite from './assets/plusWhite.svg';

const data = ref([{
  type: 'tile',
  title: '',
  content: [
    {
      link: 'https://lalala1.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala155.ru',
      icon: iconPatch
    }
  ],
},
{
  type: 'row',
  title: '',
  content: [
    {
      link: 'https://lalala2.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala3.ru',
      icon: iconPatch
    }
  ],
},
{
  type: 'favorite',
  title: '',
  content: [
    {
      link: 'https://lalala10.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala11.ru',
      icon: iconPatch
    }
  ],
}
]);

const popinOpened = ref(false);
const selectPopinOpened = ref(false);
const inputValue = ref('');
const currentIndex = ref(0);
const addInput = ref(null);

const openPopin = (index) => {
  selectPopinOpened.value = false;
  popinOpened.value = true;
  currentIndex.value = index;
}
watch(popinOpened, (value) => {
  setTimeout(() => {
    if (value && addInput.value) {
      addInput.value.focus();
    }
  }, 100);
});

const onPopinButtonClick = () => {
  if (inputValue.value.length > 0) {
    data.value[currentIndex.value].content.push({
      link: inputValue.value,
      icon: iconPatch
    });
  }
  inputValue.value = '';
  popinOpened.value = false;
}

const onSelectButtonClick = (type) => {
  data.value.push({ type, content: [] });
  selectPopinOpened.value = false;
}

const changeCardTitle = (payload) => {
  data.value[payload.index].title = payload.title;
}
</script>

<template>
  <div class="wrapper">
    <div class="container">

      <Card v-for="(item, index) in data" :key="index" :type="item.type" :index="index" :title="item.title"
        :content="item.content" @addLink="openPopin(index)" @cangeTitle="changeCardTitle" />
      <button class="selectButton" @click="selectPopinOpened = true; popinOpened = false;" style="--span: 3">
        <img :src="plusIconWhite" alt="">
      </button>

    </div>
    <div v-if="popinOpened" class="popin">
      <input type="text" v-model="inputValue" class="popinInput" ref="addInput">
      <button class="popinButton" @click="onPopinButtonClick">Add</button>
    </div>
    <div v-if="selectPopinOpened" class="popin">
      <button class="popinButton select" @click="onSelectButtonClick('tile')">Tile</button>
      <button class="popinButton select" @click="onSelectButtonClick('row')">Row</button>
      <button class="popinButton select" @click="onSelectButtonClick('favorite')">Faworite</button>
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

  @media screen and (max-width: 1023px) {
    padding-bottom: 0.56vw;
    margin-bottom: 1vw;
  }
}

.cardContent {
  display: grid;
  gap: 0.35vw;

  @media screen and (max-width: 1023px) {
    gap: 1vw;
  }
}

.cardContent_type_tile {
  grid-template-columns: repeat(8, 1fr);
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
  margin-bottom: 0.34vw;
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
  margin-bottom: 0.34vw;
  color: #ffffff;
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
  background: #D9D9D9;
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
  background: #D9D9D9;
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
