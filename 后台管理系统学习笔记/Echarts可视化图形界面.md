[TOC]

## Echarts可视化图形界面

### 1. 简单使用

1. npm install echarts --save
2. 引入echarts
3. 通过echarts.init(dom,theme,options)进行初始化生产实例
4. 通过实例setOption()方法,传入一个option数据设置绘制的图像

**在setup中定义ref关联一个dom元素, 是无法挂载上的, 因此要在onMounted上定义这个dom元素**

**注意要给这个dom元素分配宽度和高度**

```vue
<script lang="ts" setup>
import * as echarts from 'echarts'
import { ref, onMounted } from 'vue'
// 1.定义一个ref,通过ref拿到DOM元素
const divRef = ref<HTMLElement>()
onMounted(() => {
  // 2.当ref完成挂载关联后,才进行初始化,不然输入的是无效的DOM
  const echartsInstance = echarts.init(divRef.value!)
  // 3.option是区别表格类型的关键,也是最复杂的地方
  const option = {
    title: {
      text: 'ECharts 入门示例'
    },
    tooltip: {},
    legend: {
      data: ['销量']
    },
    xAxis: {
      data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
    },
    yAxis: {},
    series: [
      {
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      }
    ]
  }
  // 4.通过setOption完成设置
  echartsInstance.setOption(option)
})
</script>
<template>
  <div class="dashboard">
    <div ref="divRef" :style="{ width: '100%', height: '360px' }"></div>
  </div>
</template>
```



### 2.基础知识

#### 1. 渲染图表的两种方式

- SVG: 在不少场景中，SVG 具有重要的优势：它的**内存占用更低**(这对==移动端==尤其重要)、**渲染性能略高**、并且用户使用浏览器内置的**缩放功能时不会模糊**
- Canvas: Canvas 更适合绘制**图形元素数量非常大**（这一般是由数据量大导致）的图表（如热力图、地理坐标 系或平行坐标系上的大规模线图或散点图等），也利于实现某些视觉特效

简单的图无脑SVG, 复杂的图再进行考虑

