<script setup lang="ts">

const demos = [
  { title: '01-第一个 WebGL 程序：清空绘图区', path: '/demo/01' },
  { title: '02-绘制一个点', path: '/demo/02' },
  { title: '03-绘制一个点（优化代码）', path: '/demo/03' },
  { title: '04-绘制多个点', path: '/demo/04' },
  { title: '05-绘制三角形', path: '/demo/05' },
  { title: '06-动态绘制三角形', path: '/demo/06' },
  { title: '07-绘制线段', path: '/demo/07' },
  { title: '08-绘制渐变三角形（两个缓冲区）', path: '/demo/08' },
  { title: '09-绘制渐变三角形（一个缓冲区）', path: '/demo/09' },
  { title: '10-利用三角形图元绘制矩形', path: '/demo/10' },
  { title: '11-利用索引绘制矩形', path: '/demo/11' },
  { title: '12-使用三角带绘制矩形', path: '/demo/12' },
  { title: '13-使用三角扇绘制矩形', path: '/demo/13' },
  { title: '14-顶点顺序', path: '/demo/14' },
  { title: '15-绘制圆形', path: '/demo/15' },
  { title: '16-纹理贴图', path: '/demo/16' },
  { title: '17-绘制立方体', path: '/demo/17' },
  { title: '18-用计算机表示光照', path: '/demo/18' },
  { title: '19-绘制环形', path: '/demo/19' },
]

</script>

<template>
  <div>
    <p class="text-4xl w-80 mx-auto font-thin italic flex items-center justify-center">
      <simple-icons:webgl class="inline-block text-6xl mr-4" /> demos
    </p>

    <ul class="text-left mx-auto w-80">
      <li v-for="(item, index) of demos" :key="index" class="my-1 text-gray-400 hover:text-gray-600">
        <router-link :to="item.path">
          {{ item.title }}
        </router-link>
      </li>
    </ul>
  </div>
</template>

<route lang="yaml">
meta:
  layout: default
</route>
