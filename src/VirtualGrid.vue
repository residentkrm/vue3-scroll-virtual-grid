<script setup>
import { ref, onUnmounted, onMounted, computed, reactive, watch } from 'vue'
import debounce from './helpers/debounce.js'
const props = defineProps({
  data: {
    required: true,
    type: Array,
  },
  container: {
    required: true,
    type: HTMLElement
  }
})

const emit = defineEmits(['onScrollEnd'])

const pageSize = ref(0)
const innerWrapper = ref(null)
const probeItem = ref(null)
const probe = ref(null)
const offsetData = ref(0)

const metrics = reactive({
  columnsCount: 0,
  itemWidth: 0,
  itemHeight: 0,
  gapX: 0,
  gapY: 0,
  viewportHeight: 0
})

var lastEmitOn = 0
var scrollLock = false

onUnmounted(() => {
  props.container.removeEventListener('scroll', computeScrollData)
  window.removeEventListener('resize', reset)
})

onMounted(() => {
  setMetrics()
  props.container.addEventListener('scroll', computeScrollData)
  window.addEventListener('resize', reset)
})

const setMetrics = () => {
  const style = getComputedStyle(probe.value)

  metrics.gapY = parseFloat(style.rowGap)
  metrics.gapX = parseFloat(style.columnGap)
  metrics.itemWidth = probeItem.value.offsetWidth
  metrics.itemHeight = probeItem.value.offsetHeight
  metrics.columnsCount = Math.round (
    (probe.value.offsetWidth + metrics.gapX) / (metrics.itemWidth + metrics.gapX)
  )
  //metrics.viewportHeight = props.container.offsetHeight;
  metrics.viewportHeight = props.container.clientHeight;

  setMinPageSize()
  computeScrollData()
}

const setMinPageSize = () => {
  const minItems = (
    Math.ceil(
      metrics.viewportHeight / (metrics.itemHeight + metrics.gapY)) + 3
  ) * metrics.columnsCount

  pageSize.value = Math.max(minItems, pageSize.value)
}

const computeScrollData = () => {
  if (scrollLock) return;

  const a = props.container.scrollTop - innerWrapper.value.offsetTop - 50;
  const b = metrics.itemHeight + metrics.gapY;

  offsetData.value = Math.floor(
    a / b
  ) * metrics.columnsCount

  const reachedBottom = Math.ceil(props.container.scrollTop) >= bottomBound.value

  if (reachedBottom && lastEmitOn !== bottomBound.value) {
    emitScrollEnd()
  }

  const isNotBottom = props.container.scrollTop < bottomBound.value;

  if (isNotBottom && lastEmitOn === bottomBound.value) {
    lastEmitOn = 0
  }
}

const reset = () => {
  scrollLock = true
  debounce((val) => {
    setMetrics()
    scrollLock = false
  }, 1000)()
}

const emitScrollEnd = () => {
  lastEmitOn = bottomBound.value
  emit('onScrollEnd');
}

const wrapperHeight = computed(() => {
  return Math.ceil(
    props.data.length / metrics.columnsCount
  ) * (metrics.itemHeight + metrics.gapY) - metrics.gapY;
})

const wrapperPadding = computed(() => {
  return (displayStart.value/metrics.columnsCount) * (metrics.itemHeight + metrics.gapY)
})

const displayStart = computed(() => {
  return Math.max(offsetData.value, 0)
})

const displayEnd = computed(() => {
  return Math.min((displayStart.value + pageSize.value), (props.data.length))
})

const bottomBound = computed(() => {
  return innerWrapper.value.offsetTop + wrapperHeight.value - metrics.viewportHeight
})

defineExpose({
  reset
})

</script>

<template>
  <div ref="innerWrapper" :style="{
    paddingTop: wrapperPadding + 'px',
    height: wrapperHeight + 'px'
  }">
    <div class="grid gridview" ref="grid">
      <div class="probe" ref="probe">
        <div ref="probeItem">
          <slot
              name="default"
              :item="data[0]"
              :index="0"
          />
        </div>
      </div>

      <template
          v-for="index in displayEnd-displayStart"
          :key="'item' + index"
      >
        <slot
            name="default"
            :item="data[index+displayStart-1]"
            :index="index+displayStart"
        />
      </template>
    </div>
  </div>
</template>

<style scoped>
.probe {
  position: absolute;
  left: 0;
  right: 0;
  display: inherit;
  gap: inherit;
  grid-template-columns: inherit;
  opacity: 0;
  z-index: -1;
}
.grid {
  position: relative;
}
</style>
