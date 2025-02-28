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
   
      <!-- 分页组件 -->
      <div class="pagination-wrapper">
        <el-pagination 
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :page-sizes="[10, 20, 50, 100]"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
   
      <!-- 编辑弹窗 -->
      <article-edit-dialog 
        v-model="dialogVisible"
        :current-article="currentArticle"
        @refresh="fetchArticles"
      />
    </div>
  </template>
   
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import { Search, Plus } from '@element-plus/icons-vue'
  import axios from 'axios'

  // 响应式数据 
  const loading = ref(false)
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
  const currentArticle = ref(null)
   
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
   
  // 操作处理 
  const handleCreate = () => {
    currentArticle.value  = null 
    dialogVisible.value  = true 
  }
   
  const handleEdit = (row) => {
    currentArticle.value  = { ...row }
    dialogVisible.value  = true 
  }
   
  const handleDelete = async (id) => {
    try {
      await ElMessageBox.confirm(' 确认删除该文章？', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      })
      await axios.delete(`/api/articles/${id}`) 
      ElMessage.success(' 删除成功')
      fetchArticles()
    } catch (error) {
      console.error(error) 
    }
  }
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