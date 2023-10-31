<script setup>
import { ref, computed, watch, onMounted, toRefs } from 'vue'

// 轉盤css設定 數量 從父層來的資料
// const props.config = ref({
//     duration: 2000,
//     rollback: 0.3,
//     fontSize: 20,
//     width: 200,
//     height: 200,
//     gift: Array(6).fill().map((_, index) => index)
// })

const props = (defineProps({
    config: {
        type: Object,
        required: true
    },
    prize: {
        type: String,
    }
}))
const { config, prize } = toRefs(props)

// console.log(props)

const isRunning = ref(false)
const currentDeg = ref(0)
const targetDeg = ref(0)
const giftsDeg = ref([])
const randomRollbackdeg = ref(1)

const rotate = computed(() => {
    return 360 / config.value.gift.length
})

const translateZ = computed(() => {
    return (config.value.height / 2) / Math.tan((rotate.value / 2 / 180) * Math.PI)
})

const autoTurn = (target) => {
    let rotationToTarget; // 從當前位置到目標位置所需的旋轉
    if (target > 0 && target <= config.value.gift.length) {
        console.log('autoTurn target: ', target);
        const currentGiftIndex = (targetDeg.value / rotate.value) % config.value.gift.length; // 當前的禮物索引
        rotationToTarget = ((target - 1 - currentGiftIndex + config.value.gift.length) % config.value.gift.length) * rotate.value;
    } else {
        console.log('autoTurn', 'random');
        rotationToTarget = Math.floor(Math.random() * config.value.gift.length) * rotate.value;
    }

    const fullRotation = 360 * 5 + rotationToTarget; // 5圈的旋轉 + 需要到達目標位置的旋轉
    targetDeg.value += fullRotation;
    console.log('targetDeg', targetDeg.value % 360)
    randomRollbackdeg.value = config.value.rollback ? (Math.random() + 0.5) / 2 * config.value.rollback + 1 : 1;
    isRunning.value = true;
}

const emits = defineEmits(['finished'])

const autoTurnStop = () => {
    currentDeg.value = targetDeg.value % 360 // 結束時的角度 == 目前角度
    // 顯示獎品資料(結束角度 + 單片角度/2)
    let giftName = null
    const endDeg = currentDeg.value + (rotate.value / 2)
    giftsDeg.value.forEach((gift) => {
        if (endDeg >= gift.from && endDeg <= gift.to) {
            giftName = gift.name
        }
    })
    isRunning.value = false
    // console.log('finished',giftName)
    // console.log("autoTurnStop")
    emits('finished', giftName);
}

const logGiftsDeg = () => {
    config.value.gift.forEach((gift, index) => {
        giftsDeg.value[index] = {
            from: index === 0 ? 0 : giftsDeg.value[index - 1].to,
            to: index === 0 ? rotate.value : giftsDeg.value[index - 1].to + rotate.value,
            name: index
        }
    })
}

watch(isRunning, (newValue) => {
    setTimeout(() => {
        autoTurnStop()
    }, config.value.duration + 200)
})

onMounted(() => {
    logGiftsDeg()
    // console.log('giftsDeg', giftsDeg.value)
})

//test area
watch(prize, (newValue) => {
    const target = config.value.gift.find((item) => {
        return item.id === newValue
    })
    console.log('target', target.id)
    autoTurn(Number(target.id))
})

let timer = null;
let timeout = null;


const test = (value, index) => {
    console.log('test', value, index);
 
    if (timer) {
        clearInterval(timer);
        timer = null;
    }
    if (timeout) {
        clearTimeout(timeout);
        timeout = null;
    }

    timer = setInterval(() => {
        targetDeg.value += 100;
    }, 100);

    timeout = setTimeout(() => {
        testStop(value);
    }, 5000);
};


const testStop = (value) => {
    autoTurn(value);
    clearInterval(timer);
    clearTimeout(timeout);
};


defineExpose({
    autoTurn,
    autoTurnStop,
    test,
    testStop
})

// const currentGiftIndex = (targetDeg.value / rotate.value) % config.value.gift.length; // 當前的禮物索引
// rotationToTarget = ((target - 1 - currentGiftIndex + config.value.gift.length) % config.value.gift.length) * rotate.value;
//  targetDeg.value % 360
//  targetDeg.value / config.value.gift.length
// 
// 1: {from: 0, to: 60, id: 0}
// 2: {from: 60, to: 120, id: 1}
// 3: {from: 120, to: 180, id: 2}
// 4: {from: 180, to: 240, id: 3} 
// 5: {from: 240, to: 300, id: 4}
// 6: {from: 300, to: 360, id: 5}
</script>

<template>
    <!-- 轉盤 -->
    <div class="gift-container duration-[2000ms]"
        :style="{ 'transform': `rotateX(-${targetDeg}deg)`, 'transition-timing-function': `cubic-bezier(0.1, 0, 0, ${randomRollbackdeg})` }">
        <div class="gift border border-black" v-for="(value, index) in config.gift" :key="index"
            :style="`transform: rotateX(${rotate * index}deg) translateZ(${translateZ}px)`">
            {{ value.id }}
        </div>
    </div>
    <!-- 按鈕 -->
    <div class="flex space-y-2 flex-col w-20">
        <button class="bg-gray-400 py-3 px-4 ms-1 mt-1 rounded" @click="() => autoTurn()">start</button>
        <template v-for="(value, index) in config.gift" :key="index">
            <button class="bg-gray-400 py-3 px-4 ms-1 mt-1 rounded" @click="test(value, index)">
                {{ value.id }}
            </button>
        </template>
    </div>
</template>

<style scoped>
.gift-container {
    user-select: none;
    position: relative;
    display: flex;
    align-items: center;
    transform-style: preserve-3d;
    width: 250px;
}

.gift {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    /**props.config */
    width: 200px;
    /**props.config */
    height: 200px;
    background-color: #fff;
    /**props.config */
    font-size: 20px;
    left: 10px
}

.gift img {
    padding: 1px;
}
</style>
