<template>
  <div class="dashboard-container">
    <div>
      <el-button icon="el-icon-bell" type="text" style="margin-right: 8px;color: #000000;" size="normal" @click="goChat" />
    </div>
    <div class="dashboard-editor-container">
      <panel-group />
      <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:0px;">
        <line-chart />
      </el-row>
    </div>
  </div>
</template>

<script>
import PanelGroup from './dashboard/PanelGroup'
import LineChart from './dashboard/LineChart'
import { count } from '@/api/monitor/visits'

/**
 * 记录访问，只有页面刷新或者第一次加载才会记录
 */
count().then(res => {
})

export default {
  name: 'Dashboard',
  components: {
    PanelGroup,
    LineChart
  },
  methods: {
    goChat() {
      this.$router.push('/chat')
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .dashboard-editor-container {
    padding: 18px 22px 22px 22px;
    background-color: rgb(240, 242, 245);
    .chart-wrapper {
      background: #fff;
      padding: 16px 16px 0;
      margin-bottom: 32px;
    }
  }
</style>
