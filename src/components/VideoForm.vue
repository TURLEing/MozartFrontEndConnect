<template>
    <div class="form">
  
      <!-- 上传图片模块 -->
      <div class="upload-img">
        <span class="tips"><b>请选择视频</b></span>
        <input class="imgInput" type="file" id="file" multiple @change="handleFileChange">
        <!-- 视频预览 -->
        <!-- 这段先空着，希望将来提取出视频封面放在这里 -->
        <!-- <div v-if="ImgChosen">
          <img :src="ImgURL" alt=""  class = "showImg">
        </div> -->
        <div>
            <!-- <div v-if="MediaList&&MediaList[0]&&MediaList[0].url">
                <video :src="MediaList[0].url" style="width:320px;height:200px;" controls="controls" id="upvideo">您的浏览器不支持视频播放</video>
            </div> --> 
            <canvas id="mycanvas" style="display: none"></canvas>
        </div>

        
      </div>
  
      <!-- 选择模式模块 -->
      <div class="options">
        <div class="select">
          <span class="ModeTips"><b>请选择模式</b></span>
          <el-select v-model="selectedMode" placeholder="请选择模式">
            <el-option v-for=" item  in  modes " :key="(item.value as number)" :label="(item.mode as string)"
               :value="(item.value as number)">{{ item.mode }}
            </el-option>        
          </el-select>
        </div>
      </div>
  
      <!-- 选择时长模块 -->
      <div class="text">
        <span class="tips"><b>请输入期望音频时长</b></span>
        <div class="textbox">
          <input type="text" v-model="textInput" placeholder="输入一个数字，单位为秒">
        </div>
        
      </div>
  
      <!-- 提交模块 -->
      <div class="submit">
        <!-- 提交按钮 -->
        <el-button  class="submit-btn" type="primary" @click="handleClick" :disabled="isLoading"> 
          <span v-if="!isLoading">提交</span>
          <span v-else>处理中…</span> 
          <img v-if="isLoading" class = 'loading' src="/Loading.svg"/>
          <span v-if="isLoading"> 。 </span>  
          <el-icon v-if="!isLoading">
            <Upload/>
          </el-icon>
        </el-button>
      </div>
  
    </div>
  </template>
  
  <script lang="ts" setup>
  import {Upload, UploadFilled, Watch} from '@element-plus/icons-vue';
  import { useStore } from 'vuex';
  import { ElMessage, ElMessageBox } from 'element-plus';
  //Loading动画的css样式
  import 'animate.css';
  import {ref, onUnmounted} from 'vue';
  
  const store = useStore();
  const MediaList = ref<File[]>([]); // 声明一个 ref，初始化为空数组
  const selectedMode = ref<Number>()
  const textInput = ref('');
  const emit = defineEmits(['update:modelValue'])
  //决定Loading动画是否展示
  const isLoading = ref(false);
  let ImgChosen = ref(false);
  //提取封面url
  const getFileUrl = (file: File) => {
    return URL.createObjectURL(file);
  };
  let ImgURL = ref("");
  
  const modes: Array<{
    value: Number,
    mode: String
  }>
      = [{
      value: 0,
      mode: "测试用(请勿选择)"
    },
    {
      value: 1,
      mode: "MusicGen模型"
    }]
  // const modes = [
  //   { value: 0, mode: '测试用' },
  //   { value: 1, mode: 'MusicGen模型' }
  // ];
  
  const handleFileChange = (event:Event) => {
    const target = event.target as HTMLInputElement;
    const files = target.files;
    
    if (files && files.length > 0) {
      // 更新 MediaList 的值为选择的文件列表
      MediaList.value = Array.from(files);
      ImgChosen.value = true;
      ImgURL.value = MediaList.value.length > 0 ? getFileUrl(MediaList.value[0]) : ''
      console.log("视频已上传MediaList.value[0]:",MediaList.value.value[0]);
      store.commit('setImage', ImgURL.value);
    }
    console.log("封面ImgURL.value:",ImgURL.value)
  };
  
  
  
  const handleClick = async () => {
    // 一个判断，表格需要填完整才能上传，否则弹窗提醒
    if (selectedMode.value === null || textInput.value === '' || MediaList.value.length === 0) {
      window.alert('请完整填写需求');
      console.log(selectedMode.value);
      console.log(textInput.value);
      console.log(MediaList.value);
    return;
    }
    
    isLoading.value = true;
    //临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用
    emit('update:modelValue', true)
  
  // 将表单数据整合进formData
    const formData = new FormData();
    formData.append('file', MediaList.value[0]);
    formData.append('mode', selectedMode.value);
    formData.append('time', textInput.value);
  // 进行通信
    try {
      for (const pair of formData.entries()) {
            console.log(pair[0], pair[1]);  
          }
          // 下面是服务器地址和API接口
      const response = await fetch('http://localhost:3000/upload', {
        method: 'POST',
        mode: 'cors',
        body: formData,
      });
  
      if (!response.ok) {
            window.alert("请求失败，请稍后再试")
            throw new Error('Request failed due to unstable network connection');
          }
          // 获取返回的数据
          const responseData = await response.json();
  
          // 处理服务器返回的数据
          console.log(responseData);
          const prompt = ref('');
          const music = ref('');
          
          prompt.value = responseData.prompt;
          music.value = responseData.result_file;
  
          // 在这里可以根据需要进行其他操作
          
      if (response.ok) {
        // 把数据扔进vuex
        store.commit('setPrompt', prompt.value);
        store.commit('setMusic', music.value);
        // 把Exhibition中isSubmitted改成true,实现正常返回后组件变换为Music
        //临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用临时测试用
        // emit('update:modelValue', true) 
      }
     
      } catch (error) {
        console.error('Error during POST request:', error);
        ElMessage.error('An error occurred while submitting the form.');
        isLoading.value = false;
      }
      // finally {
      // // 无论成功还是失败，最终都将 loading 状态设置为 false，隐藏 loading 动画
      // isLoading.value = false;
      // }
  
      onUnmounted(()=>{
        isLoading.value = false;
        // emitter.off('getImage')
      })
  };
  </script>
  
  <style scoped>
    .form {
      font-size: 1.3em;
      width: 100%;
      height: 100%;
      padding: 0 10%;
  
      display: flex;
      flex-direction: column;
      align-items: center;
    }
  
    .showImg{
      max-height: 200px;
    }
  
    .upload-img {
      height: fit-content;
      width: 100%;    height: fit-content;
      padding: 10px;
    }
  
    .options {
      width: 100%;
      height: 30%;
      padding: 10px;
  
      display: flex;
      flex-direction: row;
      justify-content: space-between;
    }
  
    .img-url {
      height: 20%;
      width: 100%;
      padding: 10px;
  
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  
    .tips {
      margin-bottom: 15px;
      display: block;
      width: 100%;
      text-align: start;
    }
    .imgInput{
      color: rgb(22, 22, 35);
      padding: 8px 16px;
      font-size: 16px;
      border-radius: 5px;
    }
    .ModeTips {
      margin-bottom: 4px;
      display: block;
      width: 100%;
      text-align: start;
    }
  
    .select {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
    }
  
    .text {
      display: block;
      width: 100%;
      height: 20%;
      padding: 10px;
  
      display: flex;
      flex-direction: column;
      justify-content: space-between;
    }
  
    .submit {
      margin-top: 40px;
      display: block;
      width: 10%;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  
    .textbox{
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      align-items: flex-start;
    }
    
    .loading{
      width: 43%;
      height: 43%;
    }
    .submit-btn {
    background-color: rgb(135, 166, 201);
    color: rgb(255, 255, 255);
    border: none;
    padding: 8px 16px;
    font-size: 16px;
    border-radius: 5px;
    cursor: pointer;
  }
  .submit-btn:hover {
    background-color: #414f6e;
  }
  </style>