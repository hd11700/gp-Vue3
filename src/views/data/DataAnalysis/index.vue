<template>
    <div class="dashboard-container">
        <el-header>
        <h1>大学生膳食营养与健康管理系统 - 数据统计与分析</h1>
      </el-header>

      <div class="summary" style="margin-bottom: 20px;">
            <h2>数据概览</h2>
            <el-row>
              <el-col :span="8"><el-card><p>总用户数: {{ 30 }}</p></el-card></el-col>
              <el-col :span="8"><el-card><p>总膳食记录数: {{ 100 }}</p></el-card></el-col>
              <el-col :span="8"><el-card><p>平均每日摄入热量: {{ 2350 }} kcal</p></el-card></el-col>
            </el-row>
        </div>
      <!-- 顶部统计卡片 -->
      <el-row :gutter="20" style="margin-bottom: 20px;">
        <el-col :xs="24" :sm="12" :md="6" v-for="(item,index) in statsData" :key="index">
          <el-card shadow="hover">
            <div class="stat-item">
              <span class="label">{{ item.label  }}</span>
              <span class="value">{{ item.value  }}</span>
              <span class="unit">{{ item.unit  }}</span>
            </div>
          </el-card>
        </el-col>
      </el-row>
   
      <!-- 主图表区 -->
      <el-row :gutter="20">
        <!-- 营养摄入对比 -->
        <el-col :span="12">
          <el-card header="每日营养摄入对比">
            <div id="intakeChart" style="height:400px"></div>
          </el-card>
        </el-col>
        
        <!-- 饮食结构分析 -->
        <el-col :span="12">
          <el-card header="本周饮食结构分析">
            <div id="structureChart" style="height:400px"></div>
          </el-card>
        </el-col>
      </el-row>
   
      <!-- 新增学生运动结构分析 -->
      <el-row :gutter="20" style="margin-top: 20px;">
        <el-col :span="12">
          <el-card header="学生运动结构分析">
            <div id="exerciseStructureChart" style="height:400px"></div>
          </el-card>
        </el-col>
        
        <!-- 新增运动项目消耗对比 -->
        <el-col :span="12">
          <el-card header="运动项目消耗对比">
            <div id="exerciseConsumptionChart" style="height:400px"></div>
          </el-card>
        </el-col>
      </el-row>
   
      <!-- 健康指标趋势 -->
      <el-card header="健康指标趋势分析" style="margin-top:20px">
        <el-date-picker v-model="dateRange" type="daterange" style="width:300px"/>
        <div id="trendChart" style="height:350px"></div>
      </el-card>

      <!-- 新增雷达图 -->
      <el-card header="营养素摄入雷达图" style="margin-top:20px">
        <div id="radarChart" style="height:400px"></div>
      </el-card>

      <!-- 散点图 -->
      <el-card header="运动时间与BMI关系" style="margin-top:20px">
        <div id="scatterChart" style="height:400px"></div>
      </el-card>

      <!-- 堆叠柱状图 -->
      <el-card header="每日营养素摄入构成" style="margin-top:20px">
        <div id="stackedBarChart" style="height:400px"></div>
      </el-card>

      <!-- 热力图 -->
      <el-card header="食物营养成分热力图" style="margin-top:20px">
        <div id="heatmapChart" style="height:400px"></div>
      </el-card>
    </div>
  </template>
  <script>
  import * as echarts from 'echarts'
   
  export default {
    data() {
      return {
        statsData: [
          { label: '日均热量摄入', value: 2350, unit: 'kcal' },
          { label: '蛋白质达标率', value: 85, unit: '%' },
          { label: '本周运动消耗', value: 9800, unit: 'kcal' },
          { label: 'BMI指数', value: 21.5, unit: '' }
        ],
        dateRange: []
      }
    },
    mounted() {
      this.initCharts()
      this.initExerciseCharts()
    },
    methods: {
      initCharts() {
        // 营养摄入对比（柱状图）
        const intakeChart = echarts.init(document.getElementById('intakeChart')) 
        intakeChart.setOption({ 
          tooltip: { trigger: 'axis' },
          xAxis: {
            type: 'category',
            data: ['热量(kcal)', '蛋白质(g)', '脂肪(g)', '碳水(g)']
          },
          yAxis: { type: 'value' },
          series: [{
            type: 'bar',
            data: [2350, 68, 55, 320],
            itemStyle: {
              color: new echarts.graphic.LinearGradient(0,0,0,1,[ 
                { offset:0, color:'#83bff6' },
                { offset:1, color:'#188df0' }
              ])
            }
          }]
        })
   
        // 饮食结构分析（饼图）
        const structureChart = echarts.init(document.getElementById('structureChart')) 
        structureChart.setOption({ 
          tooltip: { trigger: 'item' },
          series: [{
            type: 'pie',
            radius: ['40%', '70%'],
            data: [
              { value: 35, name: '谷薯类' },
              { value: 25, name: '蛋白质' },
              { value: 20, name: '蔬果类' },
              { value: 15, name: '乳制品' },
              { value: 5, name: '零食' }
            ]
          }]
        })
   
        // 健康趋势分析（折线图）
        const trendChart = echarts.init(document.getElementById('trendChart')) 
        trendChart.setOption({ 
          tooltip: { trigger: 'axis' },
          xAxis: {
            type: 'category',
            data: ['周一','周二','周三','周四','周五','周六','周日']
          },
          yAxis: [{ type: 'value', name: '体重(kg)' }],
          series: [{
            name: 'BMI指数',
            type: 'line',
            data: [21.3,21.5,21.6,21.4,21.2,21.5,21.4],
            smooth: true 
          }]
        })
 
  // 新增雷达图初始化 
  const radarChart = echarts.init(document.getElementById('radarChart')) 
  radarChart.setOption({ 
    radar: {
      indicator: [
        { name: '蛋白质', max: 100 },
        { name: '脂肪', max: 60 },
        { name: '碳水', max: 400 },
        { name: '纤维素', max: 30 },
        { name: '维生素', max: 100 }
      ]
    },
    series: [{
      type: 'radar',
      data: [{ value: [85, 55, 320, 22, 78] }]
    }]
  })

  // 散点图初始化
  const scatterChart = echarts.init(document.getElementById('scatterChart'))
  scatterChart.setOption({
    tooltip: {},
    xAxis: {
      name: '每周运动时间 (分钟)',
      type: 'value'
    },
    yAxis: {
      name: 'BMI指数',
      type: 'value'
    },
    series: [{
      symbolSize: 20,
      data: [[30, 22.5], [60, 21.8], [90, 21.5], [120, 21.2], [150, 20.8]],
      type: 'scatter'
    }]
  })

  // 堆叠柱状图初始化
  const stackedBarChart = echarts.init(document.getElementById('stackedBarChart'))
  stackedBarChart.setOption({
    tooltip: {
      trigger: 'axis'
    },
    legend: {
      data: ['碳水化合物', '蛋白质', '脂肪']
    },
    xAxis: {
      type: 'category',
      data: ['周一', '周二', '周三', '周四', '周五']
    },
    yAxis: {
      type: 'value'
    },
    series: [
      {
        name: '碳水化合物',
        type: 'bar',
        stack: '总量',
        data: [300, 320, 280, 350, 300]
      },
      {
        name: '蛋白质',
        type: 'bar',
        stack: '总量',
        data: [60, 70, 65, 80, 75]
      },
      {
        name: '脂肪',
        type: 'bar',
        stack: '总量',
        data: [50, 40, 45, 55, 50]
      }
    ]
  })

  // 热力图初始化
  const heatmapChart = echarts.init(document.getElementById('heatmapChart'))
  const heatmapData = [
    [0, 0, 100], [0, 1, 200], [0, 2, 150],
    [1, 0, 80], [1, 1, 120], [1, 2, 90],
    [2, 0, 60], [2, 1, 100], [2, 2, 70]
  ];
  heatmapChart.setOption({
    tooltip: {
      position: 'top'
    },
    grid: {
      height: '50%',
      width: '50%',
      top: '10%'
    },
    xAxis: {
      type: 'category',
      data: ['食物A', '食物B', '食物C']
    },
    yAxis: {
      type: 'category',
      data: ['卡路里', '蛋白质', '脂肪']
    },
    visualMap: {
      min: 0,
      max: 300,
      calculable: true,
      orient: 'horizontal',
      left: 'center',
      bottom: '15%'
    },
    series: [{
      name: '热力图',
      type: 'heatmap',
      data: heatmapData,
      label: {
        show: true
      },
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        }
      }
    }]
  })
      },
      initExerciseCharts() {
        // 学生运动结构分析（饼图）
        const exerciseStructureChart = echarts.init(document.getElementById('exerciseStructureChart'));
        exerciseStructureChart.setOption({
          tooltip: { trigger: 'item' },
          series: [{
            type: 'pie',
            radius: '50%',
            data: [
              { value: 40, name: '跑步' },
              { value: 30, name: '游泳' },
              { value: 20, name: '健身' },
              { value: 10, name: '其他' }
            ],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }]
        });

        // 运动项目消耗对比（柱状图）
        const exerciseConsumptionChart = echarts.init(document.getElementById('exerciseConsumptionChart'));
        exerciseConsumptionChart.setOption({
          tooltip: { trigger: 'axis' },
          xAxis: {
            type: 'category',
            data: ['跑步', '游泳', '健身', '骑自行车']
          },
          yAxis: { type: 'value' },
          series: [{
            name: '消耗热量 (kcal)',
            type: 'bar',
            data: [500, 400, 300, 250],
            itemStyle: {
              color: '#42a5f5'
            }
          }]
        });
      }
    }
  }
  </script>