<template>
  <div class="recipe-management">
    <!-- 顶部操作栏 -->
    <div class="operation-bar">
      <el-button type="primary" @click="handleAdd">新增菜谱</el-button>
      <el-input
        v-model="searchKeyword"
        placeholder="搜索菜品..."
        style="width: 300px; margin-left: 20px"
        clearable
      />
    </div>

    <!-- 数据表格 -->
    <el-table :data="paginatedRecipes" style="width: 100%" border>
      <el-table-column prop="id" label="菜品ID" width="120" />
      <el-table-column prop="name" label="菜品名称" width="200" />
      <el-table-column label="菜品图片" width="180">
        <template #default="{ row }">
          <el-image
            style="width: 100px; height: 100px"
            :src="row.imageUrl"
            fit="cover"
          />
        </template>
      </el-table-column>
      <el-table-column prop="category" label="归属菜系" width="150" />
      <el-table-column label="操作" width="auto">
        <template #default="{ row }">
          <el-button size="small" @click="handleEdit(row)">编辑</el-button>
          <el-popconfirm
            title="确定要删除这个菜谱吗？"
            @confirm="handleDelete(row.id)"
          >
            <template #reference>
              <el-button size="small" type="danger">删除</el-button>
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页组件 -->
    <el-pagination
      @current-change="handlePageChange"
      :current-page="currentPage"
      :page-size="pageSize"
      :total="filteredRecipes.length"
      layout="total, prev, pager, next, jumper"
    />

    <!-- 新增/编辑对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="isEdit ? '编辑菜谱' : '新增菜谱'"
      width="500"
    >
      <el-form :model="form" :rules="rules" ref="formRef">
        <el-form-item label="菜品名称" prop="name">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="菜品图片" prop="imageUrl">
          <div style="display: flex; flex-direction: column; align-items: center;">
            <el-image
              v-if="form.imageUrl"
              style="width: 100px; height: 100px"
              :src="form.imageUrl"
              fit="cover"
            />
            <el-button style="margin-top: 10px;" type="primary" @click="handleUpload">点击上传</el-button>
          </div>
        </el-form-item>
        <el-form-item label="归属菜系" prop="category">
          <el-select v-model="form.category" placeholder="请选择菜系">
            <el-option
              v-for="item in cuisines"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="所含热量" prop="calories">
          <el-input v-model="form.calories" placeholder="请输入热量" />
        </el-form-item>
        <el-form-item label="所需食材" prop="ingredients">
          <el-input v-model="form.ingredients" placeholder="请输入所需食材" />
        </el-form-item>
        <el-form-item label="作用功效" prop="effect">
          <el-input v-model="form.effect" placeholder="请输入功效" />
        </el-form-item>
        <el-form-item label="适宜人群" prop="suitpeople">
          <el-input v-model="form.suitpeople" placeholder="请输入适宜人群" />
        </el-form-item>
        <el-form-item label="烹饪方法" prop="make">
          <el-input type="textarea" v-model="form.make" placeholder="请输入烹饪方法" rows="4" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="submitForm"> 确认 </el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

  // 生命周期钩子 
  onMounted(() => {
    getRecipes()
  })
// 初始示例数据
const recipes = ref([
  {
    id: 1,
    name: '鱼香肉丝',
    image: 'https://example.com/yuxiangrousi.jpg',
    cuisine: '川菜'
  },
  {
    id: 2,
    name: '宫保鸡丁',
    image: 'https://example.com/gongbaojiding.jpg',
    cuisine: '川菜'
  },
  {
    id: 3,
    name: '白切鸡',
    image: 'https://example.com/baiqieji.jpg',
    cuisine: '粤菜'
  }
])

// 表单相关状态
const dialogVisible = ref(false)
const isEdit = ref(false)
const form = ref({
  id: null,
  name: '',
  image: '',
  cuisine: ''
})
const formRef = ref(null)

// 验证规则
const rules = {
  name: [{ required: true, message: '请输入菜品名称', trigger: 'blur' }],
  image: [{ required: true, message: '请输入图片地址', trigger: 'blur' }],
  cuisine: [{ required: true, message: '请选择菜系', trigger: 'change' }]
}

// 菜系列表
const cuisines = ['川菜', '粤菜', '湘菜', '鲁菜', '浙菜', '苏菜']

// 搜索功能
const searchKeyword = ref('')

const filteredRecipes = computed(() => {
  return recipes.value.filter(item => 
    item.name.includes(searchKeyword.value) ||
    item.cuisine.includes(searchKeyword.value)
  )
})

// 分页相关状态
const currentPage = ref(1)
const pageSize = ref(10)

// 计算分页后的菜谱
const paginatedRecipes = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  return filteredRecipes.value.slice(start, start + pageSize.value)
})

// 获取菜谱数据
const getRecipes = async () => {
  const res = await axios.get('http://localhost:8080/api/recipes')
  console.log("获取菜谱数据",res.data);
  recipes.value = res.data.map((item, index) => ({
    ...item,
    id: index + 1 // 自增ID
  }))
}

// 新增处理
const handleAdd = () => {
  isEdit.value = false
  form.value = { id: null, name: '', image: '', cuisine: '' }
  dialogVisible.value = true
}

// 编辑处理
const handleEdit = (row) => {
  isEdit.value = true
  form.value = { ...row }
  dialogVisible.value = true
}

// 删除处理
const handleDelete = (id) => {
  recipes.value = recipes.value.filter(item => item.id !== id)
}

// 表单提交
const submitForm = async () => {
  await formRef.value.validate()
  
  if (isEdit.value) {
    const index = recipes.value.findIndex(item => item.id === form.value.id)
    recipes.value.splice(index, 1, form.value)
  } else {
    const newId = recipes.value.length ? Math.max(...recipes.value.map(item => item.id)) + 1 : 1
    recipes.value.push({ ...form.value, id: newId })
  }
  
  dialogVisible.value = false
}

// 处理页码变化
const handlePageChange = (page) => {
  currentPage.value = page
}

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      form.value.imageUrl = e.target.result; // 更新图片 URL
    };
    reader.readAsDataURL(file); // 读取文件为 Data URL
  }
};
</script>

<style scoped>
.recipe-management {
  padding: 20px;
}

.operation-bar {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
}

.el-image {
  border-radius: 4px;
}
</style>