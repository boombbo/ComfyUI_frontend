<template>
  <div ref="container" class="w-full h-full"></div>
</template>

<script setup lang="ts">
import { LGraphNode } from '@comfyorg/litegraph'
import { onMounted, onUnmounted, ref, toRaw, watchEffect } from 'vue'

import Load3d from '@/extensions/core/load3d/Load3d'
import Load3dAnimation from '@/extensions/core/load3d/Load3dAnimation'
import { useLoad3dService } from '@/services/load3dService'

const props = defineProps<{
  node: any
  type: 'Load3D' | 'Load3DAnimation' | 'Preview3D' | 'Preview3DAnimation'
  backgroundColor: string
  showGrid: boolean
  lightIntensity: number
  fov: number
  cameraType: 'perspective' | 'orthographic'
  showPreview: boolean
  extraListeners?: Record<string, (value: any) => void>
}>()

const container = ref<HTMLElement | null>(null)
const node = ref(props.node)
const load3d = ref<Load3d | Load3dAnimation | null>(null)

const eventConfig = {
  materialModeChange: (value: string) => emit('materialModeChange', value),
  backgroundColorChange: (value: string) =>
    emit('backgroundColorChange', value),
  lightIntensityChange: (value: number) => emit('lightIntensityChange', value),
  fovChange: (value: number) => emit('fovChange', value),
  cameraTypeChange: (value: string) => emit('cameraTypeChange', value),
  showGridChange: (value: boolean) => emit('showGridChange', value),
  showPreviewChange: (value: boolean) => emit('showPreviewChange', value)
} as const

watchEffect(() => {
  if (load3d.value) {
    const rawLoad3d = toRaw(load3d.value)

    rawLoad3d.setBackgroundColor(props.backgroundColor)
    rawLoad3d.toggleGrid(props.showGrid)
    rawLoad3d.setLightIntensity(props.lightIntensity)
    rawLoad3d.setFOV(props.fov)
    rawLoad3d.toggleCamera(props.cameraType)
    rawLoad3d.togglePreview(props.showPreview)
  }
})

const emit = defineEmits<{
  (e: 'materialModeChange', materialMode: string): void
  (e: 'backgroundColorChange', color: string): void
  (e: 'lightIntensityChange', lightIntensity: number): void
  (e: 'fovChange', fov: number): void
  (e: 'cameraTypeChange', cameraType: string): void
  (e: 'showGridChange', showGrid: boolean): void
  (e: 'showPreviewChange', showPreview: boolean): void
}>()

const handleEvents = (action: 'add' | 'remove') => {
  if (!load3d.value) return

  Object.entries(eventConfig).forEach(([event, handler]) => {
    const method = `${action}EventListener` as const
    load3d.value?.[method](event, handler)
  })

  if (props.extraListeners) {
    Object.entries(props.extraListeners).forEach(([event, handler]) => {
      const method = `${action}EventListener` as const
      load3d.value?.[method](event, handler)
    })
  }
}

onMounted(() => {
  load3d.value = useLoad3dService().registerLoad3d(
    node.value as LGraphNode,
    container.value,
    props.type
  )
  handleEvents('add')
})

onUnmounted(() => {
  handleEvents('remove')
  useLoad3dService().removeLoad3d(node.value as LGraphNode)
})

defineExpose({
  load3d
})
</script>
