<script setup>
import iconPatch from './assets/icon.png';
import plusIcon from './assets/plus.webp';
import { ref } from 'vue';

const data = ref({
  tiles: [
    {
      link: 'https://lalala1.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala155.ru',
      icon: iconPatch
    }
  ],
  rows: [
    {
      link: 'https://lalala2.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala3.ru',
      icon: iconPatch
    }
  ],
  favorites: [
    {
      link: 'https://lalala10.ru',
      icon: iconPatch
    },
    {
      link: 'https://lalala11.ru',
      icon: iconPatch
    }
  ],
});
const popinOpened = ref(false);
const addingType = ref('');
const inputValue = ref('');

const openPopin = (type) => {
  popinOpened.value = true;
  addingType.value = type;
}

const onPopinButtonClick = () => {
  if (inputValue.value.length > 0) {
    data.value[addingType.value].push({
      link: inputValue.value,
      icon: iconPatch
    });
  }
  inputValue.value = '';
  popinOpened.value = false;
}
</script>

<template>
  <div class="wrapper">
    <div class="container">
      <div class="card">
        <div class="title">Tiles-example</div>
        <div class="cardContent">
          <div v-for="item in data.tiles" :key="item.link" class="tile">
            <img :src="item.icon" alt="">
          </div>
          <div class="tile plus" @click="openPopin('tiles')">
            <img :src="plusIcon" alt="">
          </div>
        </div>
      </div>

      <div class="card">
        <div class="title">Rows-example</div>
        <div v-for="item in data.rows" :key="item.link" class="row">
          {{ item.link }}
        </div>
        <div class="row plus" @click="openPopin('rows')">
          <img :src="plusIcon" alt="">
        </div>
      </div>

      <div class="card">
        <div class="title">Favorites-example</div>
        <div v-for="item in data.favorites" :key="item.link" class="favorite">
          {{ item.link }}
        </div>
        <div class="row plus" @click="openPopin('favorites')">
          <img :src="plusIcon" alt="">
        </div>
      </div>

    </div>
    <div v-if="popinOpened" class="popin">
      <input type="text" v-model="inputValue" class="popinInput">
      <button class="popinButton" @click="onPopinButtonClick">Add</button>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  position: relative;
}

.container {
  padding-top: 20px;
  padding-left: 1vw;
  padding-right: 1vw;
  display: flex;
  gap: 1vw;
}

.card {
  width: 32vw;
  padding-top: 8px;
  padding-left: 12px;
  padding-right: 12px;
  padding-bottom: 13px;
  border-radius: 20px;
  background: #282828;
}

.cardContent {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.tile {
  width: 50px;
  height: 50px;
  border-radius: 10px;
  background: #D9D9D9;
  padding: 10px;

  img {
    display: block;
    width: 100%;
    margin: auto;
  }
}

.title {
  font-size: 12px;
  line-height: 15px;
  margin-bottom: 10px;
}

.row {
  width: calc(100% - 24px);
  border-radius: 10px;
  background: #D9D9D9;
  padding: 3.5px 10px;
  margin-bottom: 5px;
  font-size: 12px;
  color: #282828;

  img {
    display: block;
    width: 20px;
    margin: auto;
  }
}

.favorite {
  width: calc(100% - 24px);
  border-radius: 10px;
  background: #282828;
  padding: 3.5px 10px;
  margin-bottom: 5px;
  font-size: 12px;
  color: #ffffff;
}

.popin {
  position: fixed;
  top: 50%;
  left: 50%;
  padding: 20px;
  transform: translate(-50%, -50%);
  border-radius: 20px;
  background: #282828;
}

.popinInput {
  width: 500px;
  border-radius: 10px;
  background: #D9D9D9;
  padding: 3.5px 10px;
  margin-bottom: 5px;
  font-size: 12px;
  color: #282828;
  border: none;
  outline: none;
}

.popinButton {
  display: block;
  margin-top: 10px;
  border-radius: 10px;
  background: #D9D9D9;
  padding: 3.5px 10px;
  margin-bottom: 5px;
  font-size: 12px;
  color: #282828;
  border: none;
  outline: none;
}
</style>
