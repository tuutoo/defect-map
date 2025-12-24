<template>
  <div class="space-y-6 p-6 bg-gray-50 h-full overflow-y-auto">
    <h2 class="text-xl font-bold text-gray-900">参数设置</h2>

    <!-- 长方形尺寸 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">长方形尺寸</h3>
      </template>

      <div class="space-y-4">
        <UFormGroup label="宽度 (px)">
          <UInput
            v-model.number="localSettings.rectWidth"
            type="number"
            :min="10"
            :max="2000"
          />
        </UFormGroup>

        <UFormGroup label="高度 (px)">
          <UInput
            v-model.number="localSettings.rectHeight"
            type="number"
            :min="10"
            :max="2000"
          />
        </UFormGroup>
      </div>
    </UCard>

    <!-- 画布设置 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">画布设置</h3>
      </template>

      <UFormGroup label="画布扩展比例">
        <UInput
          v-model.number="localSettings.canvasExpansion"
          type="number"
          :min="1"
          :max="3"
          :step="0.1"
        />
        <template #help>
          画布大小 = 长方形尺寸 × 扩展比例
        </template>
      </UFormGroup>
    </UCard>

    <!-- 疵点设置 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">疵点颜色</h3>
      </template>

      <div class="space-y-4">
        <UFormGroup label="疵点颜色">
        <UPopover>
          <UButton :label="color" color="neutral" variant="outline">
            <template #leading>
              <span :style="chip" class="size-3 rounded-full" />
            </template>
          </UButton>

          <template #content>
            <UColorPicker v-model="color" class="p-2" />
          </template>
        </UPopover>
        </UFormGroup>

        <UFormGroup label="疵点大小 (半径px)">
          <UInput
            v-model.number="localSettings.defectSize"
            type="number"
            :min="1"
            :max="20"
          />
        </UFormGroup>
      </div>
    </UCard>

    <!-- 疵点列表 -->
    <UCard>
      <template #header>
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-semibold">疵点列表</h3>
          <UButton
            icon="i-lucide-plus"
            size="sm"
            @click="addDefect"
          >
            添加疵点
          </UButton>
        </div>
      </template>

      <div class="space-y-3">
        <div
          v-for="(defect, index) in localSettings.defects"
          :key="index"
          class="flex items-center gap-2"
        >
          <UInput
            v-model.number="defect.x"
            type="number"
            placeholder="X 坐标"
            class="flex-1"
          />
          <UInput
            v-model.number="defect.y"
            type="number"
            placeholder="Y 坐标"
            class="flex-1"
          />
          <UButton
            icon="i-lucide-trash-2"
            color="error"
            variant="ghost"
            size="sm"
            @click="removeDefect(index)"
          />
        </div>

        <div v-if="localSettings.defects.length === 0" class="text-gray-500 text-sm text-center py-4">
          暂无疵点，点击上方按钮添加
        </div>
      </div>
    </UCard>

    <!-- 批量导入 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">批量导入</h3>
      </template>

      <UFormGroup label="JSON 格式导入" help="格式: [{ x: 10, y: 20 }, ...]">
        <UTextarea
          v-model="importText"
          :rows="5"
          placeholder='[{"x": 10, "y": 20}, {"x": -15, "y": 30}]'
        />
      </UFormGroup>

      <div class="flex gap-2 mt-4">
        <UButton @click="importDefects">
          导入疵点
        </UButton>
        <UButton color="neutral" variant="ghost" @click="clearDefects">
          清空所有疵点
        </UButton>
      </div>
    </UCard>
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

const props = defineProps<{
  modelValue: Settings
}>()

const emit = defineEmits<{
  'update:modelValue': [value: Settings]
}>()

const localSettings = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const color = ref('#FF0000')
const chip = computed(() => ({ backgroundColor: color.value }))

const importText = ref('')

const addDefect = () => {
  localSettings.value = {
    ...localSettings.value,
    defects: [...localSettings.value.defects, { x: 0, y: 0 }]
  }
}

const removeDefect = (index: number) => {
  const newDefects = [...localSettings.value.defects]
  newDefects.splice(index, 1)
  localSettings.value = {
    ...localSettings.value,
    defects: newDefects
  }
}

const importDefects = () => {
  try {
    const parsed = JSON.parse(importText.value)
    if (Array.isArray(parsed)) {
      const validDefects = parsed.filter(
        (item) => typeof item.x === 'number' && typeof item.y === 'number'
      )
      localSettings.value = {
        ...localSettings.value,
        defects: validDefects
      }
      importText.value = ''
    }
  } catch (error) {
    console.error('导入失败，请检查 JSON 格式', error)
  }
}

const clearDefects = () => {
  localSettings.value = {
    ...localSettings.value,
    defects: []
  }
}
</script>
