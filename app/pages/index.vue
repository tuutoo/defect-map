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
interface Defect {
  x: number
  y: number
}

interface Settings {
  rectWidth: number
  rectHeight: number
  canvasExpansion: number
  defectColor: string
  defectSize: number
  defects: Defect[]
}

// 初始化设置
const settings = ref<Settings>({
  rectWidth: 400,
  rectHeight: 800,
  canvasExpansion: 1.4,
  defectColor: '#ff0000',
  defectSize: 3,
  defects: [
    { x: 50, y: 30 },
    { x: -70, y: -40 },
    { x: 0, y: 60 },
    { x: 100, y: -80 }
  ]
})
</script>
