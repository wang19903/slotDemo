<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import Pool from './components/Pool.vue';

const configs = ref([])
const inputNumber = ref()
const prizes = ref([])
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
</script>

<template>
  <div class="h-screen w-screen bg-gray-100">
    <h1 class="text-3xl font-bold underline">
      ﾊﾟﾁﾝｺ
    </h1>
    <div class="flex mt-5">
      <Pool v-for="(config, index) in configs" :key="index" :config="config" :prize="prizes[index]" />
    </div>

    <!-- test area -->
    <div class="mt-5 m-2">
      <p>指定號碼</p>
      <input type="number" class="py-3 px-4 rounded border" placeholder="輸入3個0~6的數字" v-model="inputNumber">
      <button class="bg-gray-400 py-3 px-4 ms-1 rounded" @click="orderHaddler">送出</button>
    </div>

  </div>
</template>

<style scoped></style>
