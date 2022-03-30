# ECharts-基础学习笔记
学习和使用过程中记录的笔记，参考链接：
- ECharts官网：[Apache ECharts](https://echarts.apache.org/zh/index.html)
- vue-echarts GitHub: [vue-echarts](https://github.com/ecomfe/vue-echarts)
- runoob.com网站教程：[ECharts 教程](https://www.runoob.com/echarts/echarts-tutorial.html)
- ECharts在线编辑工具：[ECharts在线编辑工具](https://echarts.apache.org/examples/zh/editor.html?c=doc-example/getting-started)
- runoob.com网站在线编辑：[runoob.com网站在线编辑](https://www.runoob.com/try/try.php?filename=tryecharts_intro_baidu)

## 数据传入
例如下面代码，数据在代码中已经定义了：
```vue
<template>
  <v-chart class="chart" :option="option" />
</template>

<script>
import VChart, { THEME_KEY } from "vue-echarts";
import { ref, defineComponent } from "vue";

export default defineComponent({
  name: "ProcessorPerf",
  components: {
    VChart
  },
  provide: {
    [THEME_KEY]: ""
  },
  setup () {
    const option = ref({
      title: {
      text: 'Usage Rate(%)'
      },
      tooltip: {
        trigger: 'axis'
      },
      legend: {
        data: ['user', 'sys', 'idel', 'iowait', 'entc']
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
      },
      toolbox: {
        feature: {
            saveAsImage: {}
        }
      },
      xAxis: {
        type: 'category',
        boundaryGap: false,
        data: ['01:00','02:00','03:00','04:00','05:00',
        '06:00','07:00','08:00','09:00','10:00','11:00', '12:00', 
        '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', 
        '19:00', '20:00', '21:00', '22:00', '23:00', '00:00',]
      },
      yAxis: {
        type: 'value'
      },
      series: [
        {
            name: 'user',
            type: 'line',
            smooth: true,
            data: [3,28,32,34,12,10,11,9,8,18,14,12,
            13,10,13,9,23,21,16,14,13,15,17,14]
        },
        {
            name: 'sys',
            type: 'line',
            smooth: true,
            data: [8,14,18,16,10,9,9,6,7,13,15,22,
            18,19,23,29,33,31,21,17,14,19,11,9]
        },
        {
            name: 'idel',
            type: 'line',
            smooth: true,
            data: [89,58,50,50,78,81,80,85,85,69,71,66,
            69,71,64,62,44,48,63,69,73,66,72,77]
        },
        {
            name: 'iowait',
            type: 'line',
            smooth: true,
            data: [1,7,8,6,8,5,6,3,2,3,1,1,0,3,0,2,1,3,7,4,1,4,2,1]
        },
        {
            name: 'entc',
            type: 'line',
            smooth: true,
            data: [11,42,50,50,22,19,20,15,15,31,29,34,
            31,29,36,38,56,52,37,31,27,34,28,23]
        }
      ]
    });
    return { option };
  }
});
</script>

<style scoped>
.chart {
  height: 300px;
  width: 1060px;
}
</style>
```
想把上面代码中series数据通过API获取，使用axios，
