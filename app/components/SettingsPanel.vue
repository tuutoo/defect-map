<template>
  <div class="space-y-6 p-6 bg-gray-50 h-full overflow-y-auto">
    <h2 class="text-xl font-bold text-gray-900">参数设置</h2>

    <!-- 画布设置 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">画布尺寸</h3>
      </template>

      <div class="flex gap-4">
        <UFormGroup label="宽度 (米)" :error="canvasWidthError">
          <UInput
            v-model.number="tempCanvasWidth"
            type="number"
            :min="0.1"
            :max="100"
            :step="0.01"
            @blur="validateCanvasWidth"
          />
        </UFormGroup>

        <UFormGroup label="高度 (米)" :error="canvasHeightError">
          <UInput
            v-model.number="tempCanvasHeight"
            type="number"
            :min="0.1"
            :max="100"
            :step="0.01"
            @blur="validateCanvasHeight"
          />
        </UFormGroup>
      </div>
      <template #footer>
        <p class="text-xs text-gray-500">画布尺寸必须大于长方形尺寸</p>
      </template>
    </UCard>

    <!-- 长方形尺寸 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">长方形尺寸</h3>
      </template>

      <div class="flex gap-4">
        <UFormGroup label="宽度 (米)" :error="rectWidthError">
          <UInput
            v-model.number="tempRectWidth"
            type="number"
            :min="0.1"
            :max="localSettings.canvasWidth"
            :step="0.01"
            @blur="validateRectWidth"
          />
        </UFormGroup>

        <UFormGroup label="高度 (米)" :error="rectHeightError">
          <UInput
            v-model.number="tempRectHeight"
            type="number"
            :min="0.1"
            :max="localSettings.canvasHeight"
            :step="0.01"
            @blur="validateRectHeight"
          />
        </UFormGroup>
      </div>
    </UCard>

    <!-- 疵点设置 -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">疵点设置</h3>
      </template>

      <div class="flex gap-4">
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
          <USelect
            v-model="defect.corner"
            :items="cornerOptions"
            class="w-28"
          />
          <UInput
            v-model.number="defect.x"
            type="number"
            placeholder="X (cm)"
            class="flex-1"
            :min="0"
          />
          <UInput
            v-model.number="defect.y"
            type="number"
            placeholder="Y (cm)"
            class="flex-1"
            :min="0"
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

      <UFormGroup label="JSON 格式导入" help='格式: [{ x: 10, y: 20, corner: "top-left" }, ...]'>
        <UTextarea
          v-model="importText"
          :rows="5"
          placeholder='[{"x": 10, "y": 20, "corner": "top-left"}, {"x": 15, "y": 30, "corner": "top-right"}]'
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

const toast = useToast()

// 画布尺寸的临时值和错误信息
const tempCanvasWidth = ref(props.modelValue.canvasWidth)
const tempCanvasHeight = ref(props.modelValue.canvasHeight)
const canvasWidthError = ref('')
const canvasHeightError = ref('')

// 长方形尺寸的临时值和错误信息
const tempRectWidth = ref(props.modelValue.rectWidth)
const tempRectHeight = ref(props.modelValue.rectHeight)
const rectWidthError = ref('')
const rectHeightError = ref('')

// 监听 modelValue 变化，同步临时值
watch(() => props.modelValue.canvasWidth, (newVal) => {
  tempCanvasWidth.value = newVal
})
watch(() => props.modelValue.canvasHeight, (newVal) => {
  tempCanvasHeight.value = newVal
})
watch(() => props.modelValue.rectWidth, (newVal) => {
  tempRectWidth.value = newVal
})
watch(() => props.modelValue.rectHeight, (newVal) => {
  tempRectHeight.value = newVal
})

// 校验画布宽度
const validateCanvasWidth = () => {
  canvasWidthError.value = ''
  if (tempCanvasWidth.value <= localSettings.value.rectWidth) {
    const errorMsg = '画布宽度必须大于长方形宽度'
    canvasWidthError.value = errorMsg
    toast.add({
      title: '校验失败',
      description: errorMsg,
      color: 'error',
      icon: 'i-lucide-circle-x'
    })
    tempCanvasWidth.value = localSettings.value.canvasWidth // 恢复原值
  } else {
    localSettings.value = {
      ...localSettings.value,
      canvasWidth: tempCanvasWidth.value
    }
  }
}

// 校验画布高度
const validateCanvasHeight = () => {
  canvasHeightError.value = ''
  if (tempCanvasHeight.value <= localSettings.value.rectHeight) {
    const errorMsg = '画布高度必须大于长方形高度'
    canvasHeightError.value = errorMsg
    toast.add({
      title: '校验失败',
      description: errorMsg,
      color: 'error',
      icon: 'i-lucide-circle-x'
    })
    tempCanvasHeight.value = localSettings.value.canvasHeight // 恢复原值
  } else {
    localSettings.value = {
      ...localSettings.value,
      canvasHeight: tempCanvasHeight.value
    }
  }
}

// 校验长方形宽度
const validateRectWidth = () => {
  rectWidthError.value = ''
  if (tempRectWidth.value >= localSettings.value.canvasWidth) {
    const errorMsg = '长方形宽度必须小于画布宽度'
    rectWidthError.value = errorMsg
    toast.add({
      title: '校验失败',
      description: errorMsg,
      color: 'error',
      icon: 'i-lucide-circle-x'
    })
    tempRectWidth.value = localSettings.value.rectWidth // 恢复原值
  } else {
    localSettings.value = {
      ...localSettings.value,
      rectWidth: tempRectWidth.value
    }
  }
}

// 校验长方形高度
const validateRectHeight = () => {
  rectHeightError.value = ''
  if (tempRectHeight.value >= localSettings.value.canvasHeight) {
    const errorMsg = '长方形高度必须小于画布高度'
    rectHeightError.value = errorMsg
    toast.add({
      title: '校验失败',
      description: errorMsg,
      color: 'error',
      icon: 'i-lucide-circle-x'
    })
    tempRectHeight.value = localSettings.value.rectHeight // 恢复原值
  } else {
    localSettings.value = {
      ...localSettings.value,
      rectHeight: tempRectHeight.value
    }
  }
}

const cornerOptions = [
  { value: 'top-left', label: '左上' },
  { value: 'top-right', label: '右上' },
  { value: 'bottom-left', label: '左下' },
  { value: 'bottom-right', label: '右下' }
]

const importText = ref('')

const addDefect = () => {
  localSettings.value = {
    ...localSettings.value,
    defects: [...localSettings.value.defects, { x: 0, y: 0, corner: 'top-left' as Corner }]
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
        (item) => typeof item.x === 'number' && typeof item.y === 'number' && item.corner
      ).map(item => ({
        x: item.x,
        y: item.y,
        corner: item.corner || 'top-left'
      }))
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
