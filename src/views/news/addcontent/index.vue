<template>
    <div class="article-create-container" style="padding: 20px; max-width: 800px; margin: 0 auto;">
      <!-- 标题输入 -->
      <el-input v-model="formData.title"  placeholder="请输入标题" class="title-input" style="margin-bottom: 15px;" />
      
      <!-- 封面图片上传 -->
      <el-upload 
        ref="upload"
        class="cover-upload"
        action="/api/upload"
        :on-success="handleCoverSuccess"
        :show-file-list="false"
        style="margin-bottom: 15px;"
      >
        <img v-if="formData.cover"  :src="formData.cover"  class="cover-preview" />
        <el-button v-else class="upload-icon" type="default" @click="$refs.upload.$el.click()">
          <el-icon><Plus /></el-icon> 请上传封面图片 
        </el-button>
      </el-upload>
   
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
   
  <script>
  export default {
    data() {
      return {
        formData: {
          title: '',
          content: '',
          cover: ''
        }
      }
    },
    methods: {
      // 封面图片上传成功回调 
      handleCoverSuccess(response) {
        this.formData.cover  = response.data.url  
      },
      // 提交表单 
      async submitForm() {
        try {
          await this.$axios.post('/api/article',  this.formData) 
          this.$message.success(' 文章发布成功')
          this.$router.push('/article/list') 
        } catch (error) {
          this.$message.error(' 提交失败：' + error.message) 
        }
      }
    }
  }
  </script>