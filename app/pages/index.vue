<template>
  <div class="h-screen flex flex-col">
    <!-- 标题栏 -->
    <div class="bg-gray-900 text-white p-4 shadow-lg">
      <h1 class="text-2xl font-bold">疵点图绘制工具</h1>
    </div>

    <!-- 主内容区 -->
    <div class="flex-1 flex overflow-hidden">
      <!-- 左侧画布区域 (70%) -->
      <div class="w-[70%] p-4 flex items-center justify-center overflow-auto">
        <DefectCanvas
          :canvas-width="settings.canvasWidth"
          :canvas-height="settings.canvasHeight"
          :rect-width="settings.rectWidth"
          :rect-height="settings.rectHeight"
          :defects="settings.defects"
          :defect-color="settings.defectColor"
          :defect-stroke-color="settings.defectStrokeColor"
          :defect-size="settings.defectSize"
          :selected-defect-index="settings.selectedDefectIndex"
        />
      </div>

      <!-- 右侧参数设置区域 (30%) -->
      <div class="w-[30%] border-l border-gray-200">
        <SettingsPanel v-model="settings" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
type Corner = 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'

interface Defect {
  x: number // 厘米
  y: number // 厘米
  corner: Corner
}

interface Settings {
  canvasWidth: number // 米
  canvasHeight: number // 米
  rectWidth: number // 米
  rectHeight: number // 米
  defectColor: string
  defectStrokeColor: string
  defectSize: number
  defects: Defect[]
  selectedDefectIndex: number | null
}

// 初始化设置
const settings = ref<Settings>({
  canvasWidth: 12,
  canvasHeight: 50,
  rectWidth: 8.56,
  rectHeight: 40.12,
  defectColor: '#ff0000',
  defectStrokeColor: '#000000',
  defectSize: 3,
  selectedDefectIndex: null,
  defects: [
    { x: 50, y: 30, corner: 'top-left' },
    { x: 70, y: 40, corner: 'top-right' },
    { x: 100, y: 80, corner: 'bottom-left' },
    { x: 100, y: 200, corner: 'bottom-right' },
    { x: 300, y: 500, corner: 'bottom-right' }
  ]
})
</script>
