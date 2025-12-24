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

      <div class="space-y-4">
        <div class="flex gap-4">
          <UFormGroup label="疵点颜色" class="flex-1">
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

          <UFormGroup label="疵点大小 (半径px)" class="flex-1">
            <UInput
              v-model.number="localSettings.defectSize"
              type="number"
              :min="1"
              :max="20"
            />
          </UFormGroup>
        </div>

        <UFormGroup label="疵点外框颜色">
          <UPopover>
            <UButton :label="strokeColor" color="neutral" variant="outline">
              <template #leading>
                <span :style="strokeChip" class="size-3 rounded-full" />
              </template>
            </UButton>

            <template #content>
              <UColorPicker v-model="strokeColor" class="p-2" />
            </template>
          </UPopover>
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

      <div v-if="localSettings.defects.length > 0">
        <UTable
          v-model:row-selection="rowSelection"
          :data="defectTableRows"
          :columns="defectColumns"
          @select="onSelectRow"
        >
          <template #corner-cell="{ row }">
            <div v-if="isEditingRow(row.original.id)">
              <USelect
                v-model="editingCorner"
                :items="cornerOptions"
                size="xs"
                class="w-24"
              />
            </div>
            <div v-else>
              {{ row.original.corner }}
            </div>
          </template>

          <template #x-cell="{ row }">
            <div v-if="isEditingRow(row.original.id)">
              <UInput
                v-model.number="editingX"
                type="number"
                size="xs"
                class="w-24"
              />
            </div>
            <div v-else>
              {{ row.original.x }}
            </div>
          </template>

          <template #y-cell="{ row }">
            <div v-if="isEditingRow(row.original.id)">
              <UInput
                v-model.number="editingY"
                type="number"
                size="xs"
                class="w-24"
              />
            </div>
            <div v-else>
              {{ row.original.y }}
            </div>
          </template>

          <template #action-cell="{ row }">
            <div v-if="isEditingRow(row.original.id)" class="flex gap-1">
              <UButton icon="i-lucide-check" size="xs" color="success" @click.stop="saveEdit" />
              <UButton icon="i-lucide-x" size="xs" color="neutral" variant="ghost" @click.stop="cancelEdit" />
            </div>
            <UDropdownMenu v-else :items="[
              [
                {
                  label: '编辑',
                  icon: 'i-lucide-edit',
                  onSelect: () => startEditRow(row.original.id)
                }
              ],
              [
                {
                  label: '删除',
                  icon: 'i-lucide-trash-2',
                  color: 'error',
                  onSelect: () => removeDefect(parseInt(row.original.id))
                }
              ]
            ]">
              <UButton
                icon="i-lucide-ellipsis-vertical"
                color="neutral"
                variant="ghost"
                size="xs"
              />
            </UDropdownMenu>
          </template>
        </UTable>
      </div>

      <div v-else class="text-gray-500 text-sm text-center py-4">
        暂无疵点，点击上方按钮添加
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
import type { TableColumn } from '@nuxt/ui'

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

interface RowData {
  id: string
  no: number
  corner: string
  x: number
  y: number
  actions: string
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
const strokeColor = ref('#000000')
const strokeChip = computed(() => ({ backgroundColor: strokeColor.value }))

const toast = useToast()

// 表格列定义
const defectColumns: TableColumn<RowData>[] = [
  {
    accessorKey: 'no',
    header: '序号'
  },
  {
    accessorKey: 'corner',
    header: '位置'
  },
  {
    accessorKey: 'x',
    header: 'X (cm)'
  },
  {
    accessorKey: 'y',
    header: 'Y (cm)'
  },
  {
    id: 'action',
    header: '操作'
  }
]

// 表格行数据
const defectTableRows = computed<RowData[]>(() =>
  localSettings.value.defects.map((defect, index): RowData => ({
    id: index.toString(), // 必须有 id 字段用于行选择
    no: index + 1,
    corner: cornerOptions.find(opt => opt.value === defect.corner)?.label || '',
    x: defect.x,
    y: defect.y,
    actions: '' // 占位符，用于删除按钮列
  }))
)

// 行选择状态
const rowSelection = ref<Record<string, boolean>>({})

// 编辑状态（存储正在编辑的行 ID）
const editingRowId = ref<string | null>(null)
const editingCorner = ref<Corner>('top-left')
const editingX = ref<number>(0)
const editingY = ref<number>(0)

// 监听选中状态变化
watch(rowSelection, (newSelection) => {
  const selectedIds = Object.keys(newSelection).filter(id => newSelection[id])
  if (selectedIds.length > 0 && selectedIds[0]) {
    const index = parseInt(selectedIds[0])
    localSettings.value = {
      ...localSettings.value,
      selectedDefectIndex: index
    }
  } else {
    localSettings.value = {
      ...localSettings.value,
      selectedDefectIndex: null
    }
  }
}, { deep: true })

// 处理行选择（单选）
function onSelectRow(e: Event, row: { id: string }) {
  // 如果正在编辑，不处理选择
  if (editingRowId.value) return

  const rowId = row.id
  const isSelected = rowSelection.value[rowId]

  // 清空所有选中
  rowSelection.value = {}

  // 如果之前没有选中，则选中当前行
  if (!isSelected) {
    rowSelection.value[rowId] = true
  }
}

// 开始编辑行
function startEditRow(rowId: string) {
  const index = parseInt(rowId)
  const defect = localSettings.value.defects[index]
  if (defect) {
    editingRowId.value = rowId
    editingCorner.value = defect.corner
    editingX.value = defect.x
    editingY.value = defect.y
  }
}

// 保存编辑
function saveEdit() {
  if (!editingRowId.value) return

  const index = parseInt(editingRowId.value)
  const newDefects = [...localSettings.value.defects]
  if (newDefects[index]) {
    newDefects[index] = {
      corner: editingCorner.value,
      x: editingX.value,
      y: editingY.value
    }
    localSettings.value = {
      ...localSettings.value,
      defects: newDefects
    }
  }

  cancelEdit()
}

// 取消编辑
function cancelEdit() {
  editingRowId.value = null
}

// 判断是否正在编辑
function isEditingRow(rowId: string) {
  return editingRowId.value === rowId
}

// 更新疵点数据的辅助方法
const updateDefectCorner = (index: number, value: Corner) => {
  const newDefects = [...localSettings.value.defects]
  if (newDefects[index]) {
    newDefects[index].corner = value
    localSettings.value = {
      ...localSettings.value,
      defects: newDefects
    }
  }
}

const updateDefectX = (index: number, value: number) => {
  const newDefects = [...localSettings.value.defects]
  if (newDefects[index]) {
    newDefects[index].x = value
    localSettings.value = {
      ...localSettings.value,
      defects: newDefects
    }
  }
}

const updateDefectY = (index: number, value: number) => {
  const newDefects = [...localSettings.value.defects]
  if (newDefects[index]) {
    newDefects[index].y = value
    localSettings.value = {
      ...localSettings.value,
      defects: newDefects
    }
  }
}

// 同步颜色变化
watch(color, (newColor) => {
  localSettings.value = {
    ...localSettings.value,
    defectColor: newColor
  }
})

watch(strokeColor, (newColor) => {
  localSettings.value = {
    ...localSettings.value,
    defectStrokeColor: newColor
  }
})

// 监听外部选中状态变化
watch(() => props.modelValue.defectColor, (newVal) => {
  color.value = newVal
})
watch(() => props.modelValue.defectStrokeColor, (newVal) => {
  strokeColor.value = newVal
})

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
  const newDefects = [...localSettings.value.defects, { x: 0, y: 0, corner: 'top-left' as Corner }]
  localSettings.value = {
    ...localSettings.value,
    defects: newDefects
  }
  // 自动进入新行的编辑模式
  nextTick(() => {
    const newIndex = newDefects.length - 1
    startEditRow(newIndex.toString())
  })
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
