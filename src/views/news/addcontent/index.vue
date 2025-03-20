<template>
    <div class="article-create-container" style="padding: 20px; max-width: 800px; margin: 0 auto;">
      <!-- 标题输入 -->
      <el-input v-model="formData.title"  placeholder="请输入标题" class="title-input" style="margin-bottom: 15px;" />
      
      <!-- 封面图片上传 -->
      <div style="display: flex; flex-direction: column; align-items: left;">
            <el-image
              v-if="formData.img"
              style="width: 100px; height: 100px"
              :src="formData.img"
              fit="cover"
            />
            <el-button style="margin: 10px 0 20px 0;width: 100px;" type="primary" @click="handleUpload">点击上传封面</el-button>
          </div>
   
      <!-- 内容编辑区 -->
      <el-input 
        v-model="formData.content" 
        type="textarea"
        :rows="15"
        placeholder="请输入文章内容"
        class="content-textarea"
        style="margin-bottom: 15px;"
      />
   
      <!-- 提交按钮 -->
      <el-button type="primary" @click="submitForm" >发布文章</el-button>
    </div>
  </template>
   
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import { Search, Plus } from '@element-plus/icons-vue'
  import axios from 'axios'
  import { ElMessageBox, ElMessage } from 'element-plus'
  const formData = ref({
    title: '',
    content: '',
    img: ''
  })

  const submitForm = async () => {
    try {
      await axios.post('http://localhost:8080/news', formData.value)
      ElMessage.success(' 文章发布成功')
      formData.value = {
        title: '',
        content: '',
        img: ''
      }
    } catch (error) {
      ElMessage.error(' 提交失败：' + error.message)
    }
  }

  const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onload = () => {
      const newFileName = `${formData.value.title}.jpg`;
      const newFile = new File([reader.result], newFileName, { type: file.type });
      const formData1 = new FormData();
      formData1.append('file', newFile);
      axios.post('http://localhost:8080/uploadimg', formData1, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
      .then(response => {
        formData.img = `http://localhost:8080/${newFileName}`;
      })
      .catch(error => {
        console.error("上传失败", error);
      });
    };
    reader.onerror = () => {
      console.error('文件读取失败');
    };
  }
};

const handleUpload = () => {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'image/*';
  input.onchange = handleFileChange;
  input.click();
};
  </script>