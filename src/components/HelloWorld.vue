<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import axios from "axios";
import { Pie } from "vue-chartjs"
import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    ArcElement,
    CategoryScale,
} from "chart.js"
ChartJS.register(Title, Tooltip, Legend, ArcElement, CategoryScale)

type Memory = {
    size: number,
    start: number,
    id: number,
    status: boolean
}

/** 内存的id */
const id = ref(0)
/** 内存id保存的数组 */
const idArray = ref([] as number[])
const chartData = ref({
    labels: [] as number[],
    datasets: [
        {
            backgroundColor: ['#41B883', '#E46651', '#00D8FF', '#DD1B16'],
            data: [] as number[]
        }
    ]
})
/** 接收内存分配情况的数组 */
const list = ref(Array<Memory>())

/** 分配内存的大小 */
const size = ref(0)
/** 分配算法的选择 */
const algorithm = ref(0)
/** 释放内存时接收的消息 */
const message = ref("")
/** 起始位置计算保存数组 */
const styleList = ref([] as string[])
/** 起始位置计算-位置 */
const startStyle = computed(() => {
    for (let index = 0; index < list.value.length; index++) {
        if(!idArray.value.includes(list.value[index].id)){
            idArray.value.push(list.value[index].id)
            const randomHex = Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, "0")
            styleList.value[index] = "left:" + list.value[index].start * 2 + "px;width:" + list.value[index].size * 2 + "px;background-color: #" + randomHex + ";"
        }
    }
    return styleList
})
/** 初始化 */
const init = onMounted(()=>{
    view()
})
/**
 * 分配内存空间
 * @param size 分配大小 
 * @param algorithm 使用的分配算法 1->首次适应算法 2->循环首次适应算法 3->最佳适应算法 4->最坏适应算法
 * @return Array<Memory>
 */
const allocate = (size: number, algorithm: number) => {
    axios.get("/api/allocate", {
        params: {
            size,
            algorithm
        }
    }).then((res) => {
        list.value = res.data
    }).catch((err) => {
        console.log(err)
    })
}

/**
 * 查看内存分配情况
 * @return Array<Memory>
 */
const view = () => {
    axios.get("/api/view").then((res) => {
        list.value = res.data
        for (let i = 0; i < list.value.length; i++) {
            chartData.value.labels[i] = list.value[i].id
            chartData.value.datasets[0].data[i] = list.value[i].size
        }
    }).catch((err) => {
        console.log(err)
    })
}

/**
 * 释放内存
 * @param id 要释放的内存的id
 * @return string
 */
const free = (id: number) => {
    axios.get("/api/free", {
        params: {
            id
        }
    }).then((res) => {
        message.value = res.data
        styleList.value.splice(id,1)
    }).catch((err) => {
        console.log(err);
    })
}



</script>

<template>
    <button @click="allocate(size, algorithm)">分配内存空间</button>
    <button @click="free(chartData.labels[id])">释放内存</button>
    <button @click="view()">查看内存分配情况</button>
    <input type="number" v-model="size" />
    <input type="number" v-model="algorithm" />
    <input type="number" v-model="id" />
    <!-- <Pie :chart-data="chartData" /> -->

    <!-- 条状内存状态显示 -->
    <div class="background">
        <div v-for="(item, index) in list" :key="item.id" class="each-block" :style="startStyle.value[index]">
            <div>
                {{item.size}}
            </div>
        </div>
    </div>
    {{ list }}
    {{ message }}
</template>

<style scoped>
.background {
    max-width: 1024px;
    min-width: 1024px;
    min-height: 50px;
    max-height: 50px;
    background-color: green;
    position: relative;
}

.each-block {
    position: absolute;
    height: 100%;
    line-height: 50px;
}
</style>
