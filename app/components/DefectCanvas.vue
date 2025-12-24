<template>
  <div class="relative w-full h-full bg-white flex items-center justify-center">
    <canvas
      ref="canvasRef"
      :width="canvasWidth"
      :height="canvasHeight"
      class="border border-gray-200"
    />
  </div>
</template>

<script setup lang="ts">
interface Defect {
  x: number
  y: number
}

interface Props {
  rectWidth: number
  rectHeight: number
  defects: Defect[]
  canvasExpansion: number
  defectColor: string
  defectSize: number
}

const props = withDefaults(defineProps<Props>(), {
  rectWidth: 100,
  rectHeight: 100,
  defects: () => [],
  canvasExpansion: 1.4,
  defectColor: '#ff0000',
  defectSize: 3
})

const canvasRef = ref<HTMLCanvasElement>()

// 计算画布尺寸
const canvasWidth = computed(() => Math.round(props.rectWidth * props.canvasExpansion))
const canvasHeight = computed(() => Math.round(props.rectHeight * props.canvasExpansion))

// 绘制函数
const draw = () => {
  if (!canvasRef.value) return

  const canvas = canvasRef.value
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  // 清空画布
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  // 设置白色背景
  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  // 计算长方形的位置（画布中心）
  const centerX = canvas.width / 2
  const centerY = canvas.height / 2

  // 绘制长方形
  ctx.strokeStyle = '#000000'
  ctx.lineWidth = 2
  ctx.beginPath()
  ctx.rect(
    centerX - props.rectWidth / 2,
    centerY - props.rectHeight / 2,
    props.rectWidth,
    props.rectHeight
  )
  ctx.stroke()

  // 绘制坐标系原点（可选，用于调试）
  // ctx.fillStyle = '#0000ff'
  // ctx.beginPath()
  // ctx.arc(centerX, centerY, 3, 0, Math.PI * 2)
  // ctx.fill()

  // 绘制疵点
  ctx.fillStyle = props.defectColor
  props.defects.forEach((defect) => {
    // 将直角坐标系坐标转换为画布坐标
    // 直角坐标系：原点在中心，y轴向上为正
    // 画布坐标系：原点在左上角，y轴向下为正
    const canvasX = centerX + defect.x
    const canvasY = centerY - defect.y // 注意y轴方向相反

    ctx.beginPath()
    ctx.arc(canvasX, canvasY, props.defectSize, 0, Math.PI * 2)
    ctx.fill()
  })
}

// 监听属性变化，重新绘制
watch(
  () => [
    props.rectWidth,
    props.rectHeight,
    props.defects,
    props.canvasExpansion,
    props.defectColor,
    props.defectSize
  ],
  () => {
    nextTick(() => draw())
  },
  { deep: true }
)

// 初始化时绘制
onMounted(() => {
  draw()
})
</script>
