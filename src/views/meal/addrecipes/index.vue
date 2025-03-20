<template>
    <div class="recipe-form-container" style="margin-top: 20px; margin-right: 200px;">
      <el-form :model="formData" :rules="rules" ref="formRef" label-width="120px">
        <!-- 基础信息区块 -->
        <el-form-item label="菜品名称" prop="name">
          <el-input v-model="formData.name" placeholder="请输入菜品名称"/>
        </el-form-item>
        
        <!-- 图片上传区块 -->
        <el-form-item label="菜品图片" prop="imageUrl">
          <el-upload 
            action="http://localhost:8080/uploadimg"
            :headers="headers"
            :limit="1"
            :before-upload="beforeUpload"
            :file-list="fileList">
            <el-button type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>
        
        <!-- 归属菜系 -->
        <el-form-item label="菜系分类" prop="cuisine">
          <el-select v-model="formData.cuisine" placeholder="请选择菜系">
            <el-option
              v-for="item in cuisines"
              :key="item.id"
              :label="item.name"
              :value="item.name"/>
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
  import axios from 'axios'
   
  const formData = reactive({
    name: '',
    image: '',
    calories: 0,
    cuisine: '',
    category_id: '',
    ingredients: '',
    effect: '',
    suitpeople: '',
    make: ''
  })

  const cuisines = [
    { id: "1", name: '川菜' },
    { id: "2", name: '粤菜' },
    { id: "3", name: '湘菜' },
    { id: "4", name: '鲁菜' },
    { id: "5", name: '浙菜' },
    { id: "6", name: '苏菜' }
  ]
   
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
    console.log("提交数据",formData);
    formData.imageUrl = `http://localhost:8080/${formData.name}.jpg`;
    const selected = cuisines.find(item => item.name === formData.cuisine);
    formData.category_id = selected ? selected.id : '';
    try {
      // 调用API接口提交数据
      const response = await axios.post('http://localhost:8080/api/recipes', formData)
      
    } catch (error) {
      console.log("提交失败",error);
    }
  }

  const beforeUpload = (file) => {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsArrayBuffer(file);
      reader.onload = () => {
        const newFileName = `${formData.name}.jpg`;
        const newFile = new File([reader.result], newFileName, { type: file.type });
        resolve(newFile);
      };
      reader.onerror = () => {
        reject(new Error('文件读取失败'));
      };
    });
  };

  </script>