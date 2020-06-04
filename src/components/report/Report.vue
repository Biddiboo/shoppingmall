<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <div id="main" style="width: 800px;height:400px;"></div>
    </el-card>
  </div>
</template>
<script src="echarts.min.js"></script>
<script>
import echarts from 'echarts'
import _ from 'lodash'

export default {
  data () {
    return {
    }
  },
  created () {
  },
  methods: {
  },
  async mounted () {
    const { data: res } = await this.$http.get('reports/type/1')
    var myChart = echarts.init(document.getElementById('main'))
    var option = {
      title: {
        text: '用户来源'
      },
      tooltip: {},
      legend: {},
      toolbox: {},
      grid: {},
      xAxis: [],
      yAxis: [],
      series: []
    }
    const result = _.merge(option,res.data)
    myChart.setOption(result)
  }
}
</script>
<style lang="less" scoped>
</style>
