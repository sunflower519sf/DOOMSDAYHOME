<template>
  <div class="gate-card" 
    :class="{
      'disable-stage':stage < Number(gate),
      'current-stage':stage === Number(gate),
      'complete-stage':!error && stage > Number(gate),
      'error-stage':error,
    }">
    <p class="card-title">{{title}}</p>
    
    <div class="card-content">
      <slot></slot>
      
    </div>
    
  </div>
</template>

<script setup>
import { defineProps } from "vue";


defineProps({
  title: String,
  content: String,
  gate: String,
  stage: Number,
  error: false,
})

</script>

<style scoped lang="postcss">
.gate-card{
  @apply flex flex-col items-center container bg-cool-gray-700 rounded-sm p-5;
}

.card-title{
  @apply font-bold text-gray-400 text-xl align-left text-left w-full mb-5;
}

.description{
  @apply text-gray-500 mb-5;
}

.card-content{
  @apply font-medium text-white text-left w-full;
}

.complete-stage{
  @apply border-teal-400 border-2 opacity-opacity-60;
}

.current-stage{
  @apply border-gray-400 border-2;
}

.error-stage{
  @apply border-rose-500 border-2;
}

.disable-stage{
  @apply pointer-events-none opacity-opacity-30;
}

</style>
