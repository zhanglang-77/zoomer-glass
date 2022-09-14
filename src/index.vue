<script setup>
import {
  ref,
  reactive,
  onMounted,
  onBeforeUnmount,
} from 'vue';
import { debounce } from 'lodash';

defineProps({
  imgSrc: {
    type: String,
    required: true,
  },
});

const imgRef = ref(null);
const maskVisible = ref(false);

const maskPosition = reactive([0, 0]);

const containerClientRect = {
  x: 0,
  y: 0,
  w: 0,
  h: 0,
};

function getRect() {
  const { x, y, width, height } = imgRef.value.getBoundingClientRect();
  containerClientRect.x = x;
  containerClientRect.y = y;
  containerClientRect.w = width;
  containerClientRect.h = height;
}

const debounceGetRect = debounce(getRect, 500);

onMounted(() => {
  getRect();
  window.addEventListener('scroll', debounceGetRect);
});

onBeforeUnmount(() => {
  window.removeEventListener('scroll', debounceGetRect);
});

function onMouseMove(e) {
  const x = e.clientX - containerClientRect.x - .25 * containerClientRect.w;
  const y = e.clientY - containerClientRect.y - .25 * containerClientRect.h;

  maskPosition[0] = x;
  maskPosition[1] = y;

  if (x < 0) {
    maskPosition[0] = 0;
  }

  if (x > .5 * containerClientRect.w) {
    maskPosition[0] = .5 * containerClientRect.w;
  }

  if (y < 0) {
    maskPosition[1] = 0;
  }

  if (y > .5 * containerClientRect.h) {
    maskPosition[1] = .5 * containerClientRect.h;
  }
}

function onMouseLeave() {
  maskVisible.value = false;
}

function onMouseEnter(e) {
  maskVisible.value = true;
}
</script>

<template>
  <div ref="imgRef" class="zoomer" @mousemove="onMouseMove" @mouseleave="onMouseLeave" @mouseenter="onMouseEnter">
    <div class="img">
      <img :src="imgSrc" alt="" />
    </div>
    <div v-show="maskVisible" class="mask"
         :style="`transform: translate(${ maskPosition[0] }px, ${ maskPosition[1] }px);`" />
    <div v-show="maskVisible" class="zoom" @mousemove.stop>
      <img :style="`transform: translate(${ -maskPosition[0] * 2 }px, ${ -maskPosition[1] * 2 }px);`" :src="imgSrc"
           alt="" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.zoomer {
  width: 100%;
  height: 100%;
  position: relative;

  .mask {
    width: 50%;
    height: 50%;
    background-color: rgba(0, 0, 0, 0.3);
    position: absolute;
  }

  .img {
    width: 100%;
    height: 100%;
    overflow: hidden;
    position: absolute;

    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
  }

  .zoom {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    right: calc(-100% - 16px);
    overflow: hidden;

    img {
      width: 200%;
      height: 200%;
      object-fit: cover;
    }
  }

}
</style>

