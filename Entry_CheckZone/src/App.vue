<template>
  <p class="text-4xl text-cool-gray-400 m-10">SHELTER ZONE - 🚧 邊境檢查</p>

  <div class="gate-column">
    <Gate title="Client Information">
      <p>IP: <span class="text-teal-400">{{clientInfo.ip}}</span></p>
      <p>Country: <span class="text-teal-400">{{clientInfo.country}}</span></p>
    </Gate>
  </div>

  <hr class="border-gray-400 sm:w-2/4 w-3/4 mt-4 mb-4">
  
  <!-- Gate 1-2 -->
  <div class="gate-row mb-2">
    <Gate title="Gate 01 - 加入伺服器" class="ms:mr-1.5" gate="1" :stage="formData.curGate">
      <p class="description">請先加入Discord伺服器</p>
      <a class="text-cool-gray-700 bg-teal-400 px-4 rounded-sm"
        :href="serverLink"
        target="_blank"
        
        @click="formData.curGate = 2"
      >
        Join Server
      </a>
    </Gate>

    <Gate title="Gate 02 - 來源管道" class="sm:ml-1.5" 
      gate="2" 
      :stage="formData.curGate" 
      :error="gateError.source">
      <p class="description">從何來到此處</p>
      <div class="btn-container">
        <div class="btn"
          v-for="source, index in formData.inviteSource" 
          :key="source.name"
          :class="{selected : source.selected}" 
          @click="selectSource(index)"
          >
          {{source.name}}
        </div>
      </div>
    </Gate>
  </div>

  <!-- Gate 3-4 -->
  <div class="gate-column">
    <Gate title="Gate 03 - 避難者條款" 
      gate="3" 
      class="mb-2"
      :stage="formData.curGate"
      :error="gateError.terms">
      <p class="description">請依照提示重複輸入文字</p>
      
      <div class="input-container">
        <input type="text" 
          v-for="term, index in formData.terms" :key="term.placeholder"
          :placeholder="term.placeholder" 
          v-model="formData.terms[index].value"
          class="form-input"
          :class="{
            'error-input':formData.terms[index].value !== formData.terms[index].placeholder,
            'right-input':formData.terms[index].value === formData.terms[index].placeholder
          }"
          >
      </div>

      <hr class="mb-2 border-gray-400">

      <div class="other-notice mb-5">
        <p class="text-lg mb-2">其他注意事項:</p>
        <p>1. 不要在任何分享性質頻道閒聊與問答</p>
        <p>2. 除非問題涉及隱私，否則別亂私訊他人</p>
        <p>3. 拜託，當個有素質的正常人</p>
      </div>

        <p>如違反以上規定，且不聽從管理員勸告指示者，一律永Ban</p>

        <div class="checkbox-container">
          <input type="checkbox" id="agree" 
            v-model="formData.agree" 
            class="mx-5"
            @change="termsCheck">
          <label for="agree" >我同意以上條款</label>
        </div>
    </Gate>

    <Gate title="Gate 04 - 認證碼" class="mb-2" 
      gate="4" 
      :stage="formData.curGate">
      <p class="description">於伺服器 💾terminal 頻道輸入 atid 指令可獲取自己的Discord ID</p>
      <p class="description">請確實輸入ID，否則產生的驗證碼為無效驗證碼</p>
      <input type="text" 
        class="form-input"
        v-model="formData.userID"
        placeholder="此輸入你的 Discord ID" >
        <button @click="generateToken">產生驗證碼</button>
      <p v-if="token !== ''">你的驗證碼: <span class="text-orange-400">{{token}}</span></p>
      <p v-if="cooling.isCooling" class="text-teal-400">{{cooling.msg}}</p>
      <p class="text-gray-400 mt-2">於伺服器 💾terminal 頻道輸入指令 p\check + 你的驗證碼</p>
      <p class="text-cyan-500">範例: p\check 8@2f89%2</p>
    </Gate>

    <p class="text-gray-400">如有任何指令或驗證碼無法運作，請聯繫管理員或Proladon#7525</p>

    <Dialog v-if="clientInfo.error" @manualCountry="updateCountry" />

  </div>

</template>

<script setup>
import ipLocation from 'iplocation'
import axios from 'axios'
import { onMounted, reactive, ref } from 'vue';
import evnData from './assets/evnData.json'
import Gate from '/src/components/Gate.vue'
import Dialog from '/src/components/Dialog.vue'


//:: Data
const serverLink = ref("")
const token = ref("")
const cooling = reactive({
  coolDown: 10,
  isCooling: false,
  msg: '',
})

const clientInfo = reactive({
  ip: "Loading...",
  country: "Loading...",
  error: false
})


const formData = reactive({
  curGate: 1,
  inviteSource: [
    {
      name: '巴哈文章',
      selected: false
    },
    {
      name: 'Youtube',
      selected: false
    },
    {
      name: '朋友介紹',
      selected: false
    },
    {
      name: 'Google',
      selected: false
    }
  ],
  source: '',
  userID: '',
  user:{
    name:String,
    discriminator: String,
    bot: Boolean
  },
  terms:[
    {
      placeholder: '禁止引戰/謾罵/洗頻',
      value: '',
    },
    {
      placeholder: '減少不雅用詞',
      value: '',
    },
    {
      placeholder: '尊重與遵從管理員指示',
      value: '',
    },
    {
      placeholder: '保持頻道主題相符話題',
      value: '',
    }
  ],
  agree: false
})


const gateError = reactive({
  source: false,
  terms: false,
})


//:: Gate Logic
// 選擇來源
const selectSource = (index)=>{
  formData.inviteSource.forEach(source => source.selected = false);
  const current = formData.inviteSource[index].selected
  formData.inviteSource[index].selected = !current
  
  formData.source = formData.inviteSource[index].name
  
  formData.curGate = 3

  sourceCheck()
}


// 產生驗證碼
const generateToken = ()=>{
  const id = formData.userID
  const country = clientInfo.country.trim()
  const source = formData.source

  if (country === 'Loading...') {
    clientInfo.error = true
    return
  }

  const check = idCheck()
  if(!check || cooling.isCooling) return


  token.value = "認證碼產生中...請稍後"

  // Call Encode API
  axios.post(evnData.encodeAPI, {
    data:{
      country,
      id,
      source
    }
  })
  .then(res=>{
    console.log(res.data)
    token.value = res.data
    throttle() // 請求發送油門 (避免大量請求)
  })
  .catch(err=>{
    console.log(err)
    token.value = "伺服器錯誤，請聯絡管理員"
  })
}


//:: Gate Check
const sourceCheck = ()=>{
  let count = 0
  
  for(const source of formData.inviteSource){
    if(source.selected) count += 1
  }

  if(count === 0){
    gateError.source = true
  } else{
    gateError.source = false
  }
}


const termsCheck = ()=>{
  for(const term of formData.terms){
    const text = term.value.trim()
    if(text === '' || text !== term.placeholder){
      gateError.terms = true
      formData.agree = false
      return
    }else{
      gateError.terms = false
      formData.curGate = 4
    }
  }
}


const idCheck = ()=>{
  const userID = formData.userID.trim()

  if(userID === ''){
    token.value = "請輸入你的 Discord ID"
    return false
  }
  else if(isNaN(+userID)){
    token.value = "ID 格式錯誤"
    return false
  }
  else if(userID.length < 18){
    token.value = "ID 格式錯誤"
    return false
  }
  else{
    return true
  }
}


//:: Utils
// 取得使用者IP地址國家
const getClientInfo = () => {
  axios.get('https://api.ipify.org?format=json')
    .then(async res => {
      clientInfo.ip = res.data.ip
      try{
        const ipData = await ipLocation(res.data.ip)
        clientInfo.country = ipData.country.name
      }catch (err){
        console.log(err)
        clientInfo.err = err
      }
    })
    .catch(err=>{
      console.log(err)
      clientInfo.err = true
    })
}


// 手動選擇國家
const updateCountry = (e) => {
  clientInfo.country = e
  clientInfo.error = false
}


// 請求發送油門 (避免大量請求)
const throttle = ()=>{
  console.log("cooling...")
  cooling.isCooling = true
  let count = cooling.coolDown
  
  // 倒數 cooldown 時間
  for(let i=0; i < cooling.coolDown; i++){
    setTimeout(()=>{
      count -= 1
      if(count === 0){
        cooling.msg = ''
        cooling.isCooling = false
      }else{
        cooling.msg = `冷卻中...${count}`
      }
    }, i*1000)
  }
}



onMounted(()=>{
  getClientInfo();

  axios.get(evnData.szData)
    .then(res=>{
      serverLink.value = res.data['invite_link']
    })
})
</script>

<style lang="postcss">
body{
  @apply bg-cool-gray-800;
}

#app{
  @apply flex flex-col justify-center items-center mb-20;
}

.gate-row{
  @apply flex flex-col w-3/4 sm:flex-row sm:w-2/4;
}

.gate-column{
  @apply flex flex-col items-center justify-center w-3/4 sm:w-2/4;
}

.form-input{
  @apply text-teal-400 bg-cool-gray-700 focus:border-b-1  focus:border-cool-gray-400 focus:text-gray-100 outline-none  px-2 m-2;
}

.error-input{
  @apply border-b-1 border-rose-500 text-rose-500;
}

.right-input{
  @apply border-b-1 border-teal-400;
}

.input-container{
  @apply flex flex-wrap justify-around sm:block mb-5;
}

.other-notice{
  @apply text-gray-400;
}

.btn{
  @apply text-gray-400 border-gray-400 border-1 px-4 w-1/2 cursor-pointer text-center select-none;
}

.btn-container{
  @apply flex flex-wrap w-3/4 w-full;
}

.checkbox-container{
  @apply border-gray-400 border-1 text-center mt-5 rounded-sm p-1;
}

.selected{
  @apply bg-teal-400 border-teal-400 text-cool-gray-700;
}

.description{
  @apply text-gray-500 mb-5;
}




</style>