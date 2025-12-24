<template>
  <div class="relative w-full h-full bg-white flex items-center justify-center">
    <canvas
      ref="canvasRef"
      :width="canvasPixelWidth"
      :height="canvasPixelHeight"
      class="border border-gray-200"
    />
  </div>
</template>

<script setup lang="ts">
type Corner = 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'

interface Defect {
  x: number // 厘米
  y: number // 厘米
  corner: Corner
}

interface Props {
  canvasWidth: number // 米
  canvasHeight: number // 米
  rectWidth: number // 米
  rectHeight: number // 米
  defects: Defect[]
  defectColor: string
  defectSize: number
}

const props = withDefaults(defineProps<Props>(), {
  canvasWidth: 10,
  canvasHeight: 50,
  rectWidth: 8.56,
  rectHeight: 40.12,
  defects: () => [],
  defectColor: '#ff0000',
  defectSize: 5
})

const canvasRef = ref<HTMLCanvasElement>()

// 计算画布的像素尺寸 - 高度占满容器，宽度按比例
const canvasPixelHeight = ref(800)
const canvasPixelWidth = computed(() => {
  // 根据画布的米数宽高比计算像素宽度
  const aspectRatio = props.canvasWidth / props.canvasHeight
  return Math.round(canvasPixelHeight.value * aspectRatio)
})

// 更新画布高度以适应容器
const updateCanvasSize = () => {
  if (canvasRef.value) {
    const parent = canvasRef.value.parentElement
    if (parent) {
      canvasPixelHeight.value = parent.clientHeight - 40 // 减去一些边距
    }
  }
}

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

  // 计算比例：像素/厘米（基于画布尺寸）
  const pixelPerCmWidth = canvasPixelWidth.value / (props.canvasWidth * 100) // 米转厘米
  const pixelPerCmHeight = canvasPixelHeight.value / (props.canvasHeight * 100)

  // 使用较小的比例以保持纵横比
  const pixelPerCm = Math.min(pixelPerCmWidth, pixelPerCmHeight)

  // 计算长方形在画布上的实际像素尺寸
  const rectPixelWidth = props.rectWidth * 100 * pixelPerCm // 米转厘米再转像素
  const rectPixelHeight = props.rectHeight * 100 * pixelPerCm

  // 长方形在画布上居中
  const rectLeft = (canvas.width - rectPixelWidth) / 2
  const rectTop = (canvas.height - rectPixelHeight) / 2

  // 绘制长方形
  ctx.strokeStyle = '#000000'
  ctx.lineWidth = 2
  ctx.beginPath()
  ctx.rect(rectLeft, rectTop, rectPixelWidth, rectPixelHeight)
  ctx.stroke()

  // 绘制疵点
  ctx.fillStyle = props.defectColor
  props.defects.forEach((defect) => {
    let canvasX: number, canvasY: number

    // 根据角落位置计算坐标（相对于画布的四个角）
    switch (defect.corner) {
      case 'top-left':
        canvasX = defect.x * pixelPerCm
        canvasY = defect.y * pixelPerCm
        break
      case 'top-right':
        canvasX = canvas.width - defect.x * pixelPerCm
        canvasY = defect.y * pixelPerCm
        break
      case 'bottom-left':
        canvasX = defect.x * pixelPerCm
        canvasY = canvas.height - defect.y * pixelPerCm
        break
      case 'bottom-right':
        canvasX = canvas.width - defect.x * pixelPerCm
        canvasY = canvas.height - defect.y * pixelPerCm
        break
    }

    ctx.beginPath()
    ctx.arc(canvasX, canvasY, props.defectSize, 0, Math.PI * 2)
    ctx.fill()
  })
}

// 监听属性变化，重新绘制
watch(
  () => [
    props.canvasWidth,
    props.canvasHeight,
    props.rectWidth,
    props.rectHeight,
    props.defects,
    props.defectColor,
    props.defectSize,
    canvasPixelHeight.value
  ],
  () => {
    nextTick(() => draw())
  },
  { deep: true }
)

// 初始化时绘制和监听窗口大小变化
onMounted(() => {
  updateCanvasSize()
  draw()

  window.addEventListener('resize', () => {
    updateCanvasSize()
    draw()
  })
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', updateCanvasSize)
})
</script>
