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
          :rect-width="settings.rectWidth"
          :rect-height="settings.rectHeight"
          :defects="settings.defects"
          :canvas-expansion="settings.canvasExpansion"
          :defect-color="settings.defectColor"
          :defect-size="settings.defectSize"
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
  rectWidth: number // 米
  rectHeight: number // 米
  canvasExpansion: number
  defectColor: string
  defectSize: number
  defects: Defect[]
}

// 初始化设置
const settings = ref<Settings>({
  rectWidth: 8.56,
  rectHeight: 40.12,
  canvasExpansion: 1.2,
  defectColor: '#ff0000',
  defectSize: 5,
  defects: [
    { x: 50, y: 30, corner: 'top-left' },
    { x: 70, y: 40, corner: 'top-right' },
    { x: 100, y: 80, corner: 'bottom-left' },
    { x: 60, y: 50, corner: 'bottom-right' }
  ]
})
</script>
