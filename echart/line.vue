<!--
 * @Description: 企业对比 - 中标图表
 * @LastEditors: gzw
 * @Date: 2019-03-14 18:40:11
 * @LastEditTime: 2019-06-06 17:25:06
 -->
<template>
  <div class="chart" :style="{height: chartHeight + 'px'}">
    <v-chart :options="chartOptions" ref="charts" :autoresize="true"/>
  </div>
</template>

<script>
import ECharts from "vue-echarts";
import "echarts/lib/chart/line"; // 折线图
import "echarts/lib/component/tooltip"; // 提示
import "echarts/lib/component/legend"; // 图例
import "echarts/lib/component/grid"; // 直角坐标系
import companyInfoMixin from "@/mixins/companyInfoMixin";
import store from "@/store";
import { parseTime } from "@/utils/index";

var dataLength = 30;

const legendColors = [
      "#c23531",
      "#2f4554",
      "#61a0a8",
      "#d48265",
      "#91c7ae",
      "#749f83",
      "#ca8622",
      "#bda29a",
      "#6e7074",
      "#546570",
      "#c4ccd3"];

export default {
  name: "bidNum",
  mixins: [companyInfoMixin],
  props: [
    'x', // 日期数据
    'y', // 中标数据
    'company' // 图例公司数据
  ],
  data() {
    return {
      chartHeight: 280, // 图表高度控制
      chartOptions: {  // echarts图表配置, 详见https://echarts.baidu.com/option.html
        legend: {
            data: [],
            bottom: '8%',
            orient: 'vertical',
            align: 'auto',
            formatter: name => {
              return name.length > 20 ? name.substring(0 ,20) + '...' : name
            }
        },
        tooltip: {
          trigger: "axis",
          confine: true,
          axisPointer: { type: "cross", label: { show: false, backgroundColor: "#6a7985" } },
          formatter: function(params) {
            let tpl = '';
            let arr = params.map(item => {
              return `${item.marker} ${item.seriesName}: ${item.value[1]}`;
            });
            let _date = parseTime(params[0].axisValue);
            _date = dataLength < 30 ? _date.substring(0,7) : _date.substring(5,10);
            tpl = `${_date} <br /> ${arr.join('<br />')}`;
            return tpl;
          }
        },
        grid: { top: 20, left: '12%', bottom: 40, right: '7%', height: 200 },
        xAxis: [
          {
            type: "time",
            boundaryGap: false,
            minInterval: 3600 * 24 * 1000 * 7,
            axisLabel: {
              fontSize: 10,
              formatter: (value, index) => { // x轴月份去年
                let _date = parseTime(value);
                return dataLength < 30 ? _date.substring(0,7) : _date.substring(5,10)
              }
            },
            splitLine: { show: false },
          }
        ],
        yAxis: [ { type: "value" } ],
        series: []
      }
    };
  },
  watch: {
    /**
     * @description: 监听图表中标数量变化,重新渲染图表
     * @param {type} 
     * @return: 
     */
    y() {
      this.chartOptions.xAxis[0].data = this.x;
      let arr = Object.keys(this.y).map(key => {
        dataLength = this.y[key].length;
        return {
          name: key,
          type: "line",
          showSymbol: false,
          areaStyle: {},
          data: this.y[key]
        }
      });
      this.chartHeight = 270 + arr.length * 30; // 图表容器高度重新计算
      this.chartOptions.series = arr;
      this.chartOptions.legend.data = this.company;
      this.$nextTick(() => {
        this.$refs.charts.resize(); // 图表高度重新计算
      });
    }
  },
  components: {
    "v-chart": ECharts
  }
};

</script>

<style lang="less" scoped>
.chart {
  width: 100%;
  height: 100%;
  margin-bottom: 15px;
}
.echarts {
  width: 100%;
  height: 100%;
}
</style>
