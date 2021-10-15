<template>
   <div class="dialog-wrapper">
       <div class="dialog-container">
           <p class="title">選擇國家</p>
           <p class="description">伺服器無法偵測到你的國家，請手動選擇</p>
            <select name="" id="" @change="selectCountry($event.target.value)">
                <option v-for="country in countrysList" :key="country" 
                            :value="country" 
                            >
                    {{country}}
                </option>
            </select>
            <div class="confirm-btn" @click="confirm">確認</div>
       </div>
   </div>
</template>

<script setup>
import { defineEmit, ref } from 'vue'
import countrys from '../assets/countrys.json'
const countrysList = countrys.countrys
const selectedCountry = ref(countrys.countrys[0])

const emit = defineEmit(['manualCountry'])

// confirm
const selectCountry = (country) => {
    selectedCountry.value = country
}
const confirm = () => {
    emit('manualCountry', selectedCountry.value)
}

</script>

<style scoped lang="postcss">
.title{
    @apply text-gray-400 text-2xl;
}

.dialog-wrapper{
    @apply grid place-items-center;
    @apply fixed top-0 bottom-0 left-0 right-0 p-5;
    @apply bg-gray-900 bg-opacity-80;
}

.dialog-container{
    @apply border p-5;
    @apply bg-gray-800 border-gray-500 rounded-sm;
}

select{
    @apply w-1/2;
}

.confirm-btn{
    @apply border px-2 mt-5;
    @apply cursor-pointer text-center text-gray-400 border-teal-400 rounded-sm;
    @apply hover: text-gray-800 hover: bg-teal-400;
}
</style>