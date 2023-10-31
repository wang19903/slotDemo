<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import Pool from './components/Pool.vue';
const childRefs = ref([])
const configs = ref([])
const inputNumber = ref(0)
const prizes = ref([])
const isRunning = ref(false)
const isStopping = ref(false)
onMounted(() => {
  // mock api 池子資料
  fetch('/src/ConfigApi.json')
    .then((res) => res.json())
    .then((res) => {
      configs.value = res.config
      // console.log(configs.value)
    }).catch((err) => {
      console.log(err)
    })
  // 後端抽獎結果API...
})

const orderHaddler = () => {
  // test area
  if (inputNumber.value === undefined || inputNumber.value.length < 111 || inputNumber.value > 999) {
    return alert('請輸入3個0~6的數字')
  }
  prizes.value = String(inputNumber.value).split('')
}


const getRandomNumber = (min, max) => {
  return Math.floor(Math.random() * (max - min + 1)) + min;
};

const start = () => {
  if (isRunning.value) {
    return
  }
  isRunning.value = true;

  const randomNumbers = [
    getRandomNumber(1, 5),
    getRandomNumber(1, 5),
    getRandomNumber(1, 5)
  ];

  childRefs.value.forEach((child, index) => {
    if (child.test && randomNumbers[index] !== undefined) {
      child.test(randomNumbers[index]);
    }
  });
};

const testStop = () => {
  console.log('testStop',isStopping.value)
  if (isStopping.value) {
    return
  }

  isStopping.value = true;
  childRefs.value.forEach((child) => {
    child.testStop();
  });
  finished()
};
const finished = (value) => {
  isRunning.value = false;
  isStopping.value = false;
  console.log('finished', value)
}
</script>

<template>
  <div class="h-screen w-screen bg-gray-100">
    <div class="flex mt-5">
      <Pool v-for="(config, index) in configs" :key="index" :config="config" :prize="prizes[index]" ref="childRefs"
        @finished="finished($event)" />
    </div>

    <!-- test area -->
    <div class="mt-5 m-2">
      <p>指定號碼</p>
      <input type="number" class="py-3 px-4 rounded border" placeholder="輸入3個0~6的數字" v-model="inputNumber">
      <button class="bg-gray-400 py-3 px-4 ms-1 rounded" @click="orderHaddler">送出</button>
    </div>
    <button class="bg-gray-400 py-3 px-4 ms-1 rounded" @click="start">start</button>
    <button class="bg-gray-400 py-3 px-4 ms-1 rounded" @click="testStop">stop</button>
  </div>
</template>

<style scoped></style>
