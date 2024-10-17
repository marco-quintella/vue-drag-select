<script setup lang="ts">
import { computed, ref } from 'vue';


class DOMVector {
  constructor(
    readonly x: number,
    readonly y: number,
    readonly magnitudeX: number,
    readonly magnitudeY: number
  ) {
    this.x = x
    this.y = y
    this.magnitudeX = magnitudeX
    this.magnitudeY = magnitudeY
  }

  toDOMRect(): DOMRect {
    return new DOMRect(
      Math.min(this.x, this.x + this.magnitudeX),
      Math.min(this.y, this.y + this.magnitudeY),
      Math.abs(this.magnitudeX),
      Math.abs(this.magnitudeY),
    )
  }
}

const items = Array.from({ length: 30 }, (_, i) => String(i))

const dragVector = ref<DOMVector>()
const selectionRect = computed(() => dragVector.value?.toDOMRect())

const selectedItems = ref<Record<string, boolean>>({})
const containerRef = ref<HTMLElement>()

function intersect(rect1: DOMRect, rect2: DOMRect) {
  if (rect1.right < rect2.left || rect2.right < rect1.left) return false
  if (rect1.bottom < rect2.top || rect2.bottom < rect1.top) return false
  return true
}

function updateSelectedItems(dragVector: DOMVector) {
  if (!containerRef.value) return

  const next: Record<string, boolean> = {}
  const containerRect = containerRef.value.getBoundingClientRect()

  containerRef.value.querySelectorAll('[data-item]').forEach(el => {
    if (!containerRef.value || !(el instanceof HTMLElement)) return

    const itemRect = el.getBoundingClientRect()
    const x = itemRect.x - containerRect.x
    const y = itemRect.y - containerRect.y
    const translatedItemRect = new DOMRect(
      x,
      y,
      itemRect.width,
      itemRect.height
    )

    if (!intersect(dragVector.toDOMRect(), translatedItemRect)) return
    if (el.dataset.item && typeof el.dataset.item === 'string')
      next[el.dataset.item] = true
  })

  selectedItems.value = next
}

function onPointerDown(e: PointerEvent) {
  if (e.button !== 0 || !(e.currentTarget instanceof HTMLElement)) return

  const containerRect = e.currentTarget.getBoundingClientRect()

  dragVector.value = new DOMVector(
    e.clientX - containerRect.x,
    e.clientY - containerRect.y,
    0,
    0
  )

  e.currentTarget.setPointerCapture(e.pointerId)
}

function onPointerMove(e: PointerEvent) {
  if (!dragVector.value || !(e.currentTarget instanceof HTMLElement)) return

  const containerRect = e.currentTarget.getBoundingClientRect()

  const nextDragVector = new DOMVector(
    dragVector.value.x,
    dragVector.value.y,
    e.clientX - containerRect.x - dragVector.value.x,
    e.clientY - containerRect.y - dragVector.value.y
  )

  dragVector.value = nextDragVector
  updateSelectedItems(dragVector.value)
}

function onPointerUp() {
  dragVector.value = undefined
}

const cStyle = computed(() => ({
  top: `${selectionRect.value?.y}px`,
  left: `${selectionRect.value?.x}px`,
  width: `${selectionRect.value?.width}px`,
  height: `${selectionRect.value?.height}px`
}))
</script>

<template>
  <div class="select-none">
    <div class="absolute top-0">
      {{ selectionRect }}
    </div>

    <div class="flex justify-between">
      <div class="p-2 b-2 b-b-0 b-black b-solid">Selectable Area</div>
      <div class="p-2 b-2 b-b-0 b-black b-solid">Count: {{ Object.keys(selectedItems).length }}</div>
    </div>

    <div ref="containerRef" @pointerdown="onPointerDown" @pointermove="onPointerMove" @pointerup="onPointerUp"
      class="relative z-0 p-4 grid grid-cols-8 sm:grid-cols-10 gap-4 b-2 b-solid b-black">
      <div v-for="item of items" :key="item" :data-item="item" :class="{
        'bg-black text-white': !!selectedItems[item]
      }" class="b-2 b-black b-solid size-10 flex justify-center items-center text-center hover:bg-pink-600">
        {{ item }}
      </div>

      <div v-if="selectionRect" class="absolute b-2 b-solid b-black bg-black bg-op-30" :style="cStyle" />
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
