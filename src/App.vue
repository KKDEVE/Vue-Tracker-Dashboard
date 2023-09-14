<script setup>
import {ref, shallowRef, computed, watch, nextTick} from  'vue'
import Chart from 'chart.js/auto'


const weights = ref([
  {weight: 60.0,
    date: new Date().getTime()}
])
const weightChartEl = ref(null)
const weightChart = shallowRef(null)
const weightInput = ref(60.0)
const currentWeight = computed(()=>{
  return weights.value.sort((a,b)=>b.date-a.date)[0]||{
    weight: 0
  }
})
const addWeight=()=>{
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })
}
const removeWeight = (index) => {
  weights.value.splice(index, 1)
}

watch(weights, newWeights =>{
  const ws = [...newWeights]
  if (weightChart.value){
    weightChart.value.data.labels = 
    ws.sort((a,b)=> a.date - b.date).map(w=>new Date(w.date).toLocaleDateString())
    .slice(-30)

    weightChart.value.data.datasets[0].data = 
    ws.sort((a,b)=> a.date - b.date).map(w=> w.weight)
    .slice(-30)

    weightChart.value.update()
    return
  }

  nextTick(()=>{
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'),{
      type: "line",
      data:{
        labels: ws.sort((a,b)=> a.date - b.date).map(w=>new Date(w.date).toLocaleDateString()),
      datasets:[
          { label:"Weight",
            data:ws.sort((a,b)=> a.date - b.date).map(w=> w.weight),
            borderColor:"#CFB8FF",
            borderWidth: 1,
            fill: true,
          }  ]
      },
      options:{responsive:true,
      maintainAspectRatio: false,
    }
    })
  })
  console.log(ws)

}, {deep: true})

const hoveredElement = ref(null);

const isHovered = (element) => {
  return hoveredElement.value === element;
};

const setHovered = (element) => {
  hoveredElement.value = element;
};

</script>

<template>
  <main>
  <header>
    <div class="container-header">
    <div class="logo">
      <img src="/asset/logo.png" class="logo-img" alt="">
      <div class="logo-text">DATA</div>
    </div>
    <div class="on">
      <div class="on-text">ON</div>
      <img src="/asset/down.png" class="on-img" alt="">
    </div>
  </div>
  </header>
   <div class="buttons">
    <button class="button" :class="{ active: isHovered('input') }" >
      <img :src="isHovered('input') ? '/asset/arrow-white.png' : '/asset/arrow.png'" class="arrow" alt="">
      INPUT
    </button>
    <button class="button" :class="{ active: isHovered('chart') }">
      <img :src="isHovered('chart') ? '/asset/arrow-white.png' : '/asset/arrow.png'" class="arrow" alt="">
      CHART
    </button>
    <button class="button" :class="{ active: isHovered('history') }" >
      <img :src="isHovered('history') ? '/asset/arrow-white.png' : '/asset/arrow.png'" class="arrow" alt="">
      RECORD
    </button>
   </div>
   <div class="input" @mouseover="setHovered('input')" @mouseout="setHovered(null)">
    <div class="current">
      <div class="weight-num">{{currentWeight.weight}}</div>
    </div>
     <form @submit.prevent="addWeight" class="form">
      <input type="number"
      step="0.1"  v-model="weightInput" class="form-input"/>
      <input type="submit" value="Add data" class="form-submit"/>
     </form>
   </div>
  

   <div v-if="weights && weights.length > 0" >
    <div class="container-chart"  @mouseover="setHovered('chart')" @mouseout="setHovered(null)"> 
      <h3 class="chart-title">Last 30 days</h3>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
    </div>
    <div class="weight-history" @mouseover="setHovered('history')" @mouseout="setHovered(null)">
      <h3>
        Data History
      </h3>
      <ul class="history-lists">
        <li v-for="weight in weights" class="history-list">
          <div>
            {{weight.weight}}
          </div>
          <small>
            {{new Date(weight.date).toLocaleDateString()}}
          </small>
          <img @click="removeWeight" src="/asset/delete.png" class="history-delete" alt="">
      
        </li>
      </ul>
    </div>
   </div>
  </main>

</template>


<style >
</style>
