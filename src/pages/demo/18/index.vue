<script setup lang="ts">
import { deg2radians } from '@3dgl/math'
import { identity, multiply, ortho, rotationX, rotationY } from '@3dgl/matrix'
import { createProgram, createShader, getWebGLContext } from '@3dgl/utils'
import README from './README.md'

function hex2rgb(hex: string) {
  return hex
    .replace(
      /^#([0-9a-zA-Z]{2})([0-9a-zA-Z]{2})([ 0-9a-zA-Z ]{2})$/g,
      (match, r, g, b) => {
        return [r, g, b].map(item => parseInt(item, 16)).join(',')
      },
    )
    .split(',')
    .map(item => Number(item) / 255)
}
const canvas = ref<HTMLCanvasElement>(null!)
const ambient = ref(0.3)
const lightColor = ref('#ffffff')
onMounted(() => {
  const gl = getWebGLContext(canvas.value)

  /*
   * 顶点着色器
   */
  const VERTEX_SHADER_SOURCE = `
    precision mediump float;
    attribute vec3 a_Position;
    attribute vec4 a_Color;
    varying vec4 v_Color;
    uniform mat4 u_Matrix;
    void main() {
      gl_Position = u_Matrix * vec4(a_Position, 1);
      v_Color = a_Color;
      gl_PointSize = 5.0;
    }
  `

  /**
   * 定义片元着色器
   */
  const FRAG_SHADER_SOURCE = `
    precision mediump float;
    varying vec4 v_Color;
    uniform vec3 u_LightColor;
    uniform float u_AmbientFactor;
    void main() {
      vec3 ambientColor = u_AmbientFactor * u_LightColor;
      gl_FragColor = vec4(ambientColor, 1) * v_Color;
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
  const a_Color = gl.getAttribLocation(program, 'a_Color')
  const u_Matrix = gl.getUniformLocation(program, 'u_Matrix')

  /**
   * 光照数据
   */
  const u_AmbientFactor = gl.getUniformLocation(program, 'u_AmbientFactor')
  const u_LightColor = gl.getUniformLocation(program, 'u_LightColor')

  watchEffect(() => {
    gl.uniform1f(u_AmbientFactor, ambient.value)
    const [r, g, b] = hex2rgb(lightColor.value)
    gl.uniform3f(u_LightColor, r, g, b)
  })

  /**
   * 顶点数据
   */
  const positions = [
    /**
     * 前面
     */
    -0.5, -0.5, 0.5, 1, 0, 0, 1, 0.5, -0.5, 0.5, 1, 0, 0, 1, 0.5, 0.5, 0.5, 1,
    0, 0, 1, -0.5, 0.5, 0.5, 1, 0, 0, 1,

    /**
     * 左面
     */
    -0.5, 0.5, 0.5, 0, 1, 0, 1, -0.5, 0.5, -0.5, 0, 1, 0, 1, -0.5, -0.5, -0.5,
    0, 1, 0, 1, -0.5, -0.5, 0.5, 0, 1, 0, 1,

    /**
     * 右面
     */
    0.5, 0.5, 0.5, 0, 0, 1, 1, 0.5, -0.5, 0.5, 0, 0, 1, 1, 0.5, -0.5, -0.5, 0,
    0, 1, 1, 0.5, 0.5, -0.5, 0, 0, 1, 1,

    /**
     * 后面
     */
    0.5, 0.5, -0.5, 1, 0, 1, 1, 0.5, -0.5, -0.5, 1, 0, 1, 1, -0.5, -0.5, -0.5,
    1, 0, 1, 1, -0.5, 0.5, -0.5, 1, 0, 1, 1,

    /**
     * 上面
     */
    -0.5, 0.5, 0.5, 1, 1, 0, 1, 0.5, 0.5, 0.5, 1, 1, 0, 1, 0.5, 0.5, -0.5, 1, 1,
    0, 1, -0.5, 0.5, -0.5, 1, 1, 0, 1,

    /**
     * 下面
     */
    -0.5, -0.5, 0.5, 0, 1, 1, 1, -0.5, -0.5, -0.5, 0, 1, 1, 1, 0.5, -0.5, -0.5,
    0, 1, 1, 1, 0.5, -0.5, 0.5, 0, 1, 1, 1,
  ]

  const indices = [
    0, 1, 2, 0, 2, 3,

    4, 5, 6, 4, 6, 7,

    8, 9, 10, 8, 10, 11,

    12, 13, 14, 12, 14, 15,

    16, 17, 18, 16, 18, 19,

    20, 21, 22, 20, 22, 23,
  ]

  /**
   * 创建缓冲区
   */
  const buffer = gl.createBuffer()

  /**
   * 绑定缓冲区为当前缓冲
   */
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer)

  /**
   * 设置 a_Position 属性从缓冲区读取数据的方式
   */
  gl.vertexAttribPointer(a_Position, 3, gl.FLOAT, false, 4 * 7, 0)
  /**
   * 设置 a_Color 属性从缓冲区读取数据方式
   */
  gl.vertexAttribPointer(a_Color, 4, gl.FLOAT, false, 4 * 7, 3 * 4)
  /**
   * 向缓冲区传递数据
   */
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW)

  gl.enableVertexAttribArray(a_Position)
  gl.enableVertexAttribArray(a_Color)

  /**
   * 创建索引
   */
  const indicesBuffer = gl.createBuffer()
  /**
   * 绑定索引缓冲区
   */
  gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer)

  /**
   * 向索引缓冲区传递索引数据
   */
  gl.bufferData(
    gl.ELEMENT_ARRAY_BUFFER,
    new Uint16Array(indices),
    gl.STATIC_DRAW,
  )

  /**
   * 清空屏幕
   */
  gl.clearColor(0.0, 0.0, 0.0, 0.1)
  gl.clear(gl.COLOR_BUFFER_BIT)

  /**
   * 隐藏背面
   */
  gl.enable(gl.CULL_FACE)

  const aspect = canvas.value.width / canvas.value.height

  /**
   * 计算正交投影矩阵
   */
  const projectionMatrix = ortho({
    left: -aspect * 2,
    right: aspect * 2,
    bottom: -2,
    top: 2,
    near: 100,
    far: -100,
  })

  const dstMatrix = identity()
  const tmpMatrix = identity()

  let xAngle = 0
  let yAngle = 0

  const render = () => {
    xAngle += 1
    yAngle += 1

    // 先绕 y 轴旋转矩阵
    rotationY(deg2radians(yAngle), dstMatrix)

    // 再绕 x 轴旋转
    multiply(dstMatrix, rotationX(deg2radians(xAngle), tmpMatrix), dstMatrix)

    // 模型投影矩阵
    multiply(projectionMatrix, dstMatrix, dstMatrix)

    gl.uniformMatrix4fv(u_Matrix, false, dstMatrix)

    gl.clear(gl.COLOR_BUFFER_BIT)

    gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0)

    requestAnimationFrame(render)
  }
  render()
})
</script>
<template>
  <ShowGL title="光照" link="https://codesandbox.io/embed/18-yong-ji-suan-ji-biao-shi-guang-zhao-5ps4l?fontsize=14&hidenavigation=1&theme=dark">
    <template #canvas>
      <div class="w-full text-left">
        环境光强度:
        <input v-model="ambient" type="range" min="0" max="1" step="0.01" />
        {{ ambient }}
      </div>
      <div class="w-full text-left mb-4">
        环境光颜色：
        <input v-model="lightColor" type="color" />
        {{ lightColor }}
      </div>
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
