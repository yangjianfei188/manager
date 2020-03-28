<template>
  <el-card>
    <!-- 面包屑 -->
    <el-breadcrumb separator=">">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 图表 -->
    <div id="main" style="width: 600px;height:400px;"></div>
  </el-card>
</template>

<script>
const echarts = require("echarts");

export default {
  data() {
    return {};
  },
  mounted() {
    this.useEcharts();
  },
  methods: {
    async useEcharts() {
      //init
      var myChart = echarts.init(document.getElementById("main"));

      //获取数据
      const res = await this.$http.get(`reports/type/1`);
      console.log(res);
      let option2 = res.data.data
      // 指定图表的配置项和数据
      let option1 = {
        title: {
          text: "堆叠区域图"
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "cross",
            label: {
              backgroundColor: "#6a7985"
            }
          }
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true
        }
      };
      let option = {...option1,...option2}
      // 使用刚指定的配置项和数据显示图表。
      myChart.setOption(option);
    }
  }
};
</script>

<style>
#main {
  margin-top: 20px;
}
</style>