# ECharts-基础学习笔记
学习和使用过程中记录的笔记，参考链接：
- ECharts官网：[Apache ECharts](https://echarts.apache.org/zh/index.html)
- vue-echarts GitHub: [vue-echarts](https://github.com/ecomfe/vue-echarts)
- runoob.com网站教程：[ECharts 教程](https://www.runoob.com/echarts/echarts-tutorial.html)
- ECharts在线编辑工具：[ECharts在线编辑工具](https://echarts.apache.org/examples/zh/editor.html?c=doc-example/getting-started)
- runoob.com网站在线编辑：[runoob.com网站在线编辑](https://www.runoob.com/try/try.php?filename=tryecharts_intro_baidu)

## ECharts安装
### 直接引用
直接官网下载：[https://echarts.apache.org/zh/download.html](https://echarts.apache.org/zh/download.html)

通过标签方式直接引入构建好的`echarts`文件
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <script src="echarts.min.js"></script>
</head>
</html>
```
### 使用CDN方法
使用CDN方法引入,以jsdelivr为例：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <script src="https://cdn.jsdelivr.net/npm/echarts@4.3.0/dist/echarts.min.js"></script>
</head>
</html>
```
示例：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts示例</title>
    <!-- 引入 echarts.js -->
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.1.2/dist/echarts.min.js"></script>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 600px;height:400px;"></div>
    <script type="text/javascript">
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));
        // 指定图表的配置项和数据
        var option = {
            title: {
                text: 'ECharts Example'
            },
            tooltip: {},
            legend: {
                data:['故障量']
            },
            xAxis: {
                data: ["CPU","内存","主板","控制器","电源","硬盘","电池"]
            },
            yAxis: {},
            series: [{
                name: '故障量',
                type: 'bar',
                data: [1, 3, 0, 3, 10, 27,8]
            }]
        };
        // 使用上面配置项和数据显示图表
        myChart.setOption(option);
    </script>
</body>
</html>
```
说明：
- 注意series里面name名称要和legend中data数据值一样
- 注意数据对应关系，会根据顺序来对应

### npm获取echarts
&#8195;&#8195;例如在最近搭建的基于`@vue/cli`和`Element+`的项目中添加使用echarts。首先在项目中安装ECharts，示例如下：
```
$ npm install echarts vue-echarts
......
+ vue-echarts@6.0.0-rc.6
+ echarts@5.1.2
added 7 packages from 4 contributors and audited 1361 packages in 25.92s
```
安装成功后，在`node_modules`下就会有`echarts`和`vue-echarts`两个文件包。

Vue3中使用方法：
```js
import { createApp } from 'vue'
import ECharts from 'vue-echarts'
import { use } from "echarts/core"

// import ECharts modules manually to reduce bundle size
import {
  CanvasRenderer
} from 'echarts/renderers'
import {
  BarChart
} from 'echarts/charts'
import {
  GridComponent,
  TooltipComponent
} from 'echarts/components'

use([
  CanvasRenderer,
  BarChart,
  GridComponent,
  TooltipComponent
])

const app = createApp(...)

// register globally (or you can do it locally)
app.component('v-chart', ECharts)

app.mount(...)
```
在我的shm项目中，在`main.js`加入如下代码进行全局引用：
```js
import ECharts from 'vue-echarts'

app.component('v-chart', ECharts)
```
然后在组件中进行使用，示例代码：
```vue
<template>
  <el-row>
    <el-col :span="20" :offset="2">
      <v-chart class="chart" :option="option" />
    </el-col>
  </el-row>
</template>

<script>
import { use } from "echarts/core";
import { CanvasRenderer } from "echarts/renderers";
import { LineChart } from "echarts/charts";
import {
  TitleComponent,
  ToolboxComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent,
} from "echarts/components";
import VChart, { THEME_KEY } from "vue-echarts";
import { ref, defineComponent } from "vue";

use([
  CanvasRenderer,
  GridComponent,
  LineChart,
  TitleComponent,
  TooltipComponent,
  ToolboxComponent,
  LegendComponent
]);

export default defineComponent({
  name: "ProcessorPerf",
  components: {
    VChart
  },
  provide: {
    [THEME_KEY]: "dark"
  },
  setup () {
    const option = ref({
      title: {
      text: 'CPU使用率'
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
            saveAsImage: {test}
        }
      },
      xAxis: {
        type: 'category',
        boundaryGap: false,
        data: ['08:00','09:00','10:00','11:00', '12:00', '13:00', 
        '14:00', '15:00', '16:00', '17:00', '18:00']
      },
      yAxis: {
        type: 'value'
      },
      series: [
        {
            name: 'user',
            type: 'line',
            data: [8,18,14,12, 13, 10, 13, 9, 23, 21,16]
        },
        {
            name: 'sys',
            type: 'line',
            data: [7,13,15,22, 18, 19, 23, 29, 33, 31,21]
        },
        {
            name: 'idel',
            type: 'line',
            data: [85,69,71,66, 69, 71, 64, 62, 44, 48,63]
        },
        {
            name: 'iowait',
            type: 'line',
            data: [2,3,1,1, 0, 3, 0, 2, 1, 3,7]
        },
        {
            name: 'entc',
            type: 'line',
            data: [15,31,29,34, 31, 29, 36, 38, 56, 52,37]
        }
      ]
    });
    return { option };
  }
});
</script>

<style scoped>
.chart {
  height: 400px;
  width: 600px;
}
</style>
```
说明：
- `[THEME_KEY]`定义了图表背景颜色，示例中是深色模式
- 示例中图表主要有四个部分：
    - 标题（title）：通过`title`实现
    - 提示信息（tooltip）：鼠标移动到图表上位置，提示数据信息
    - 图例（legend）：示例中legend对应的每条折线，指出折线类型
    - X轴（xAxis）：配置X轴上的项目
    - Y轴（yAxis）：配置Y轴上的项目
    - 列表（series）：即要展示的数据的列表
- 在`series`中`type`决定图表类型，示例中是`line`
- 官方示例`series`中还有`stack`,设置成总量可以求和，示例中没有使用到

## 待补充
