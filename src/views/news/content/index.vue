<template>
    <!-- 页面容器 -->
    <div class="article-manage-container">
      <!-- 功能操作区 -->
      <div class="operation-area">
        <el-button type="primary" @click="handleCreate">
          <el-icon><Plus /></el-icon> 新增文章 
        </el-button>
        <el-input 
          v-model="searchKeyword"
          placeholder="输入标题搜索"
          style="width: 240px; margin-left: 15px"
          clearable 
        >
          <template #prefix>
            <el-icon><Search /></el-icon>
          </template>
        </el-input>
      </div>
   
      <!-- 数据表格 -->
      <el-table 
        :data="filteredArticles"
        stripe 
        style="width: 100%; margin-top: 20px"
        v-loading="loading"
      >
        <el-table-column prop="title" label="文章标题" width="360" />
        <el-table-column prop="cover" label="封面" width="120">
          <template #default="{ row }">
            <img :src="row.img" alt="封面" style="width: 100%; height: auto;" />
          </template>
        </el-table-column>
        <el-table-column prop="content" label="内容" width="500"  >
          <template #default="{ row }">
            <div style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
              {{ row.content }}
            </div>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="auto" fixed="right">
          <template #default="{ row }">
            <el-button size="small" @click="handleEdit(row)">编辑</el-button>
            <el-button 
              size="small"
              type="danger"
              @click="handleDelete(row.id)" 
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      
      <!-- 文章详情对话框 -->
      <el-dialog
        :title="isEdit ? '编辑文章' : '新增文章'"
        v-model="dialogVisible"
        width="50%"
        @close="handleClose"
      >
      <el-form :model="selectedArticle" :rules="rules" ref="formRef">
        <el-form-item label="标题" prop="title">
          <el-input
            v-model="selectedArticle.title"
            placeholder="编辑标题"
          />
        </el-form-item>
        <el-form-item label="封面" prop="img">
          <div style="display: flex; flex-direction: column; align-items: center;">
            <el-image
              v-if="selectedArticle.img"
              style="width: 100px; height: 100px"
              :src="selectedArticle.img"
              fit="cover"
            />
            <el-button style="margin-top: 10px;" type="primary" @click="handleUpload">点击上传</el-button>
          </div>
        </el-form-item>
        <el-form-item label="内容" prop="content">
          <el-input
            v-model="selectedArticle.content"
            placeholder="编辑内容"
            type="textarea"
            :autosize="{ minRows: 5 }"
          />
        </el-form-item>
      </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="handleClose">关闭</el-button>
          <el-button type="primary" @click="submitForm">保存</el-button>
        </span>
      </el-dialog>
   
      <!-- 分页组件 -->
      <div class="pagination-wrapper">
    <!-- 分页组件 -->
    <el-pagination
      @current-change="handlePageChange"
      :current-page="currentPage"
      :page-size="pageSize"
      :total="filteredArticles.length"
      layout="total, prev, pager, next, jumper"
    />
      </div>

   
    </div>
  </template>
   
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import { Search, Plus } from '@element-plus/icons-vue'
  import axios from 'axios'
  import { ElMessageBox, ElMessage } from 'element-plus'

  // 响应式数据 
  const loading = ref(false)
  const isEdit = ref(false)
  const currentPage = ref(1)
  const pageSize = ref(10)
  const total = ref(0)
  const articles = ref([
    { id: 1, title: '第一篇文章', cover: 'http://localhost:8080/减肥时的饮食.jpg', content: '这是第一篇文章的内容，内容较长，超出一行会被隐藏...' },
    { id: 2, title: '第二篇文章', cover: 'http://localhost:8080/减肥时的饮食.jpg', content: '这是第二篇文章的内容，内容较长，超出一行会被隐藏...' },
    { id: 3, title: '第三篇文章', cover: 'http://localhost:8080/减肥时的饮食.jpg', content: '这是第三篇文章的内容，内容较长，超出一行会被隐藏...' }
  ])
  const searchKeyword = ref('')
  const dialogVisible = ref(false)
  const selectedArticle = ref({
    title: '',
    img: '',
    content: ''
  })
  const rules = {
  title: [
    { required: true, message: '请输入标题', trigger: 'blur' },
    { min: 3, max: 50, message: '标题长度在 3 到 50 个字符', trigger: 'blur' }
  ],
  img: [
    { required: true, message: '请上传封面图片', trigger: 'change' }
  ],
  content: [
    { required: true, message: '请输入内容', trigger: 'blur' },
    { min: 10, message: '内容至少 10 个字符', trigger: 'blur' }
  ]
}
  const formRef = ref(null)
  // 计算属性 
  const filteredArticles = computed(() => {
    return articles.value.filter(item  => 
      item.title.includes(searchKeyword.value)  ||
      item.author.includes(searchKeyword.value) 
    )
  })
   
  // 生命周期钩子 
  onMounted(() => {
    fetchArticles()
  })

  // 处理页码变化
const handlePageChange = (page) => {
  currentPage.value = page
}
   
// 表单提交
const submitForm = async () => {
  await formRef.value.validate()
  
  if (isEdit.value) {
    const index = articles.value.findIndex(item => item.id === selectedArticle.value.id)
    articles.value.splice(index, 1, selectedArticle.value)
    try {
      // 调用API接口提交数据，URL中包含id
      const putarticles = await axios.put(`http://localhost:8080/news/${selectedArticle.value.id}`, selectedArticle.value)
      console.log("提交数据",selectedArticle.value);
    } catch (error) {
      console.log("提交失败", error);
    }
  } else {
    const newId = articles.value.length ? Math.max(...articles.value.map(item => item.id)) + 1 : 1
    articles.value.push({ ...selectedArticle.value, id: newId })
    console.log("提交数据",selectedArticle.value);
    try {
      // 调用API接口提交数据
      const response = await axios.post('http://localhost:8080/news', selectedArticle.value)
      console.log("提交数据",response);
    } catch (error) {
      console.log("提交失败",error);
    }
  }
  dialogVisible.value = false

}

  // 获取文章数据 
  const fetchArticles = async () => {
    try {
      loading.value  = true 
      // 实际项目中使用axios调用API接口 [7]()
      const res = await axios.get('http://localhost:8080/news')
      console.log("获取文章数据",res.data);
      articles.value  = res.data.data  
      total.value  = res.data.total  
    } finally {
      loading.value  = false 
    }
  }
   
  // 分页处理 
  const handleSizeChange = (newSize) => {
    pageSize.value  = newSize 
    fetchArticles()
  }
   
  const handleCurrentChange = (newPage) => {
    currentPage.value  = newPage 
    fetchArticles()
  }
   
  const handleClose = () => {
    dialogVisible.value = false
  }

  // 操作处理 
  const handleCreate = () => {
    isEdit.value = false
    selectedArticle.value = { title: '', img: '', content: '' }
    dialogVisible.value  = true 
  }
   
  const handleEdit = (row) => {
    isEdit.value = true
    selectedArticle.value = { ...row }
    dialogVisible.value = true 
  }
   
  const handleDelete = async (id) => {
    try {
      await ElMessageBox.confirm(' 确认删除该文章？', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      })
      await axios.delete(`http://localhost:8080/news/${id}`) 
      ElMessage.success(' 删除成功')
      fetchArticles()
    } catch (error) {
      console.error(error) 
    }
  }
  const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onload = () => {
      const newFileName = `${selectedArticle.value.title}.jpg`;
      const newFile = new File([reader.result], newFileName, { type: file.type });

      const formData = new FormData();
      formData.append('file', newFile);

      axios.post('http://localhost:8080/uploadimg', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
      .then(response => {
        selectedArticle.value.img = `http://localhost:8080/${newFileName}`;
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
   
  <style scoped>
  .article-manage-container {
    padding: 20px;
    background: #fff;
    border-radius: 4px;
  }
   
  .operation-area {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
  }
   
  .pagination-wrapper {
    margin-top: 20px;
    display: flex;
    justify-content: flex-end;
  }
  </style>