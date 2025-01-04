<script setup lang="ts">
import { ref } from "vue";

let rawImages = ref(
  localStorage.getItem("images") ??
    `
https://www.albertina.at/site/assets/files/3135/3_ford_falling_bough.jpg
https://i.redd.it/wldycltepvbc1.jpeg
https://i.redd.it/11l8fkanw8rc1.jpeg
https://i.redd.it/ux37hvvhstf51.jpg
`
);

const storeImages = () => {
  localStorage.setItem("images", rawImages.value);
};

const getImages = () => {
  return rawImages.value
    .split("\n")
    .map((image) => image.trim())
    .filter(Boolean);
};

let imagesIndex = ref(0);
let interval = 60000;
let transition = 4000;
let visible = ref(false);
let menuVisible = ref(false);

const getIndex = (index: number) => {
  return index % getImages().length;
};

const getImage = (index: number) => {
  return getImages()[getIndex(index)];
};

const hash = (input: string): number => {
  let hash = 0;

  for (let i = 0; i < input.length; i++) {
    const charCode = input.charCodeAt(i);
    hash = (hash * 31 + charCode) % 1000000007;
  }

  return hash < 0 ? hash + 1000000007 : hash;
};

const getAnimation = (image: string) => {
  const index = hash(image);
  const index2 = hash(`${image}random`);

  return [
    ["to-right", "to-left"][index % 2],
    ["to-bottom", "to-top"][index2 % 2],
  ];
};

const showNext = () => {
  visible.value = false;
  imagesIndex.value++;
  setTimeout(() => {
    visible.value = true;
  }, 100);
};
showNext();

setInterval(() => showNext(), interval);
</script>

<template>
  <button class="edit" @click="menuVisible = !menuVisible">edit</button>
  <textarea
    class="textarea"
    v-if="menuVisible"
    v-model="rawImages"
    cols="30"
    rows="10"
    @input="storeImages"
  ></textarea>

  <div
    :style="{
      '--transition': `${transition}ms`,
      '--interval': `${interval}ms`,
    }"
  >
    <img
      :class="['image', 'first', ...getAnimation(getImage(imagesIndex))]"
      :src="getImage(imagesIndex)"
      alt=""
    />
    <img
      :class="[
        'image',
        'second',
        { visible },
        ...getAnimation(getImage(imagesIndex + 1)),
      ]"
      :src="getImage(imagesIndex + 1)"
      alt=""
    />
  </div>
</template>

<style scoped>
.image {
  --start-x: 0%;
  --start-y: 0%;
  --end-y: 100%;
  --end-x: 100%;
  --scale: 2;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 1;
  scale: var(--scale);
  object-fit: cover;
}

.to-top {
  --start-y: 100%;
  --end-y: 0%;
}

.to-bottom {
  --start-y: 0%;
  --end-y: 100%;
}

.to-left {
  --start-x: 100%;
  --end-x: 0%;
}

.to-right {
  --start-x: 0%;
  --end-x: 100%;
}

@keyframes translate {
  0% {
    object-position: var(--start-x) var(--start-y);
    transform-origin: var(--start-x) var(--start-y);
  }
  100% {
    object-position: var(--end-x) var(--end-y);
    transform-origin: var(--end-x) var(--end-y);
  }
}

.first {
  z-index: 1;
  object-position: var(--end-x) var(--end-y);
  transform-origin: var(--end-x) var(--end-y);
}

.second {
  z-index: 2;
  opacity: 0;
}

.visible {
  opacity: 1;
  transition: var(--transition) ease;
  animation: translate var(--interval) forwards;
}

.edit {
  position: fixed;
  bottom: 0;
  right: 0;
  margin: 16px;
  z-index: 1000;
}

.textarea {
  position: fixed;
  margin: auto;
  z-index: 1000;
  width: 100%;
  max-width: 500px;
  height: 100%;
}
</style>
