<script setup lang="ts">
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
import README from './README.md'
const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (canvas.value === null) throw new Error('canvas is null')

  const gl = getWebGLContext(canvas.value)

  /**
   * 顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    precision mediump float;
    attribute vec2 a_Position;
    attribute vec2 a_Screen_Size;
    attribute vec4 a_Color;
    varying vec4 v_Color;
    void main() {
      vec2 position = (a_Position / a_Screen_Size) * 2.0 - 1.0;
      position = position * vec2(1.0, -1.0);
      gl_Position = vec4(position, 0, 1);
      v_Color = a_Color;
    }
  `
  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    varying vec4 v_Color;
    void main() {
      gl_FragColor = v_Color / vec4(255, 255, 255, 1);
    }
  `

  /**
   * 初始化着色器
   */
  const vertexShader = createShader(gl, gl.VERTEX_SHADER, VERTEX_SHADER_SOURCE)
  const fragShader = createShader(gl, gl.FRAGMENT_SHADER, FRAG_SHADER_SOURCE)

  /**
   * 初始化程序
   */
  const { program } = createProgram(gl, vertexShader, fragShader)

  gl.useProgram(program)

  /**
   * 准备数据
   */
  const a_Position = gl.getAttribLocation(program, 'a_Position')
  const a_Screen_Size = gl.getAttribLocation(program, 'a_Screen_Size')
  const a_Color = gl.getAttribLocation(program, 'a_Color')

  /**
   * 为顶点着色器中的 a_Screen_Size 传递 canvas 的宽高信息
   */
  gl.vertexAttrib2f(a_Screen_Size, canvas.value.width, canvas.value.height)

  /**
 * 存储顶点信息的数组
 */
  const positions = [
    30, 30, 255, 0, 0, 1, // v0
    30, 370, 255, 0, 0, 1, // v1
    370, 370, 255, 0, 0, 1, // v2
    370, 30, 0, 255, 0, 1, // v3
  ]

  const buffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

  /**
   * 设置 a_Position 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 4 * (2 + 4), 0)
  /**
   * 设置 a_Color 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * (2 + 4), 2 * 4)

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Color)

  /**
   * 存储顶点索引的数组
   */
  const indices = [
    0, 1, 2, // 第一个三角形
    0, 2, 3, // 第二个三角形
  ]

  /**
   * 创建索引缓冲区
   */
  const indicesBuffer = gl.createBuffer()
  /**
   * 绑定索引缓冲区
   */
  gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer)
  /**
   * 向索引缓冲区传递索引数据
   */
  gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW)
  /**
   * 清空绘制区
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)
  /**
   * 绘制
   */
  gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)
})
</script>
<template>
  <ShowGL title="利用索引绘制矩形">
    <template #canvas>
      <canvas
        ref="canvas"
        width="400"
        height="400"
        class="w-80 sm:w-[400px]"
      ></canvas>
    </template>
    <template #readme>
      <README />
    </template>
  </ShowGL>
</template>
<route lang="yaml">
meta:
  layout: empty
</route>
