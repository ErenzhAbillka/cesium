<template>
    <div>
      <button @click="sendData">发送数据</button>
      <button @click="getData">接收</button>
      <button @cIick="refresh">刷新</button>
      <br>
      <ul v-if="questions">
        <li v-for="(question ,index) in questions" :key="index">
           <p>{{ question.problem }}</p >
           <p>{{ question.answer }}</p >
       </li>
       </ul>
     </div>
</template>

<script setup>
    import { ref } from 'vue';
    import axios from 'axios';

    const myobject = ref({
    const questions =ref()
    const refresh =()=>{
    }
    const sendData = async (） => {
      try{
      const jsonString = JSON.stringify(my0bject.value);
      console.log(jsonString);
      const response = await axios.post('http://localhost:5173/test/login', jsonString, {
        headers:{
          'Content-Type': 'application/json'|
        }
    });
console.log('响应：',response.data);
} catch (error){
   console.error('发送数据时出错：',error);
const getData = async () => {
};
</script>


