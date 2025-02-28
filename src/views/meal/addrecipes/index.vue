<template>
    <div class="recipe-form-container" style="margin-top: 20px; margin-right: 200px;">
      <el-form :model="formData" :rules="rules" ref="formRef" label-width="120px">
        <!-- 基础信息区块 -->
        <el-form-item label="菜品名称" prop="name">
          <el-input v-model="formData.name" placeholder="请输入菜品名称"/>
        </el-form-item>
        
        <!-- 图片上传区块 -->
        <el-form-item label="菜品图片" prop="image">
          <el-upload 
            action="/api/upload"
            :headers="headers"
            :limit="1"
            :on-success="handleUploadSuccess"
            :file-list="fileList">
            <el-button type="primary">点击上传</el-button>
            <template #tip>支持JPG/PNG格式，建议尺寸800x600px[1]()</template>
          </el-upload>
        </el-form-item>
        
        <!-- 归属菜系 -->
        <el-form-item label="菜系分类" prop="cuisine">
          <el-select v-model="formData.cuisine" placeholder="请选择菜系">
            <el-option
              v-for="item in cuisines"
              :key="item"
              :label="item"
              :value="item"/>
          </el-select>
        </el-form-item>
        
        <!-- 所含热量 -->
        <el-form-item label="热量(kcal)" prop="calories">
          <el-input v-model="formData.calories" placeholder="请输入热量" />
        </el-form-item>
        
        <!-- 所需食材 -->
        <el-form-item label="所需食材" prop="ingredients">
          <el-input v-model="formData.ingredients" placeholder="请输入所需食材" />
        </el-form-item>
        
        <!-- 作用功效 -->
        <el-form-item label="作用功效" prop="effect">
          <el-input v-model="formData.effect" placeholder="请输入功效" />
        </el-form-item>
        
        <!-- 适宜人群 -->
        <el-form-item label="适宜人群" prop="suitpeople">
          <el-input v-model="formData.suitpeople" placeholder="请输入适宜人群" />
        </el-form-item>
        
        <!-- 制作步骤 -->
        <el-form-item label="烹饪方法" prop="make">
          <el-input type="textarea" v-model="formData.make" placeholder="请输入烹饪方法" rows="4" />
        </el-form-item>
        
        <!-- 提交按钮 -->
        <el-form-item>
          <el-button type="primary" @click="submitForm">提交菜谱</el-button>
        </el-form-item>
      </el-form>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive } from 'vue'
   
  const formData = reactive({
    name: '',
    image: '',
    calories: 0,
    cuisine: '',
    ingredients: '',
    effect: '',
    suitpeople: '',
    make: ''
  })

  // 菜系列表
const cuisines = ['川菜', '粤菜', '湘菜', '鲁菜', '浙菜', '苏菜']
   
  // 表单验证规则（参考苍穹外卖校验逻辑）
  const rules = {
    name: [{ required: true, message: '必填项', trigger: 'blur' }],
    image: [{ required: true, message: '请上传菜品图片', trigger: 'change' }],
    calories: [{ type: 'number', min: 0, message: '必须为正值' }],
    cuisine: [{ required: true, message: '请选择菜系', trigger: 'change' }],
    ingredients: [{ required: true, message: '请输入所需食材', trigger: 'blur' }],
    effect: [{ required: true, message: '请输入功效', trigger: 'blur' }],
    suitpeople: [{ required: true, message: '请输入适宜人群', trigger: 'blur' }],
    make: [{ required: true, message: '请输入烹饪方法', trigger: 'blur' }]
  }
   
  // 提交逻辑 
  const submitForm = async () => {
    try {
      await formRef.value.validate() 
      // 调用API接口提交数据（参考私人菜谱APP接口设计）[6]()
      console.log(' 提交数据:', formData)
      ElMessage.success(' 提交成功')
    } catch (error) {
      ElMessage.error(' 请完善表单')
    }
  }
  </script>