<script lang="ts" setup>
import { computed, onMounted, ref, type ComputedRef, type Ref } from 'vue'
import interact from 'interactjs'

const props = defineProps({
  id: {
    type: String,
    required: true
  },

  relativePosition: {
    type: Object,
    default: () => ({ x: 0, y: 0 })
  },

  carPosition: {
    type: Number,
    default: 0
  },

  vertical: {
    type: Boolean
  },

  large: {
    type: Boolean,
    default: false
  },

  winningCar: {
    type: Boolean,
    default: false
  }
})

const id: string = `car-${props.id}`
const debug: Ref<Boolean> = ref(true)

const setRelativePosition = (element: HTMLElement, x: number, y: number): void => {
  element.style.transform = `translate(${x}px, ${y}px)`
}

const setSize = (element: HTMLElement, large: Boolean, vertical: Boolean): void => {
  let containerWidth, containerHeight, carWidth, carHeight

  if (large) {
    containerWidth = vertical ? '150px' : '100%'
    containerHeight = vertical ? '100%' : '150px'
    carWidth = vertical ? '100%' : '325px'
    carHeight = vertical ? '325px' : '100%'
  } else {
    containerWidth = vertical ? '100px' : '100%'
    containerHeight = vertical ? '100%' : '100px'
    carWidth = vertical ? '100%' : '185px'
    carHeight = vertical ? '185px' : '100%'
  }

  element.style.width = containerWidth
  element.style.height = containerHeight
  element.style.setProperty('--car-width', carWidth)
  element.style.setProperty('--car-height', carHeight)
}

const setCarPosition = (element: HTMLElement, value: number, isVertical: boolean) => {
  if (value >= 0.2) {
    if (isVertical) {
      const carHeight = parseFloat(element.style.getPropertyValue('--car-height'))
      const carHeightPercentage = carHeight / interact.getElementRect(element).height
      const valueWithCarHeight = value - carHeightPercentage

      element.style.paddingTop = valueWithCarHeight * 100 + '%'
    } else {
      const carWidth = parseFloat(element.style.getPropertyValue('--car-width'))
      const carWidthPercentage = carWidth / interact.getElementRect(element).width
      const valueWithCarWidth = value - carWidthPercentage

      element.style.paddingLeft = valueWithCarWidth * 100 + '%'
    }
  } else {
    element.style.paddingTop = '0'
    element.style.paddingLeft = '0'
  }
  element.setAttribute('data-value', value.toFixed(2))
}

interact('#' + id).draggable({
  origin: 'self',
  inertia: true,
  listeners: {
    move(event) {
      const isVertical = event.target.getAttribute('data-orientation') === 'vertical'
      const sliderWidth = interact.getElementRect(event.target).width
      const sliderHeight = interact.getElementRect(event.target).height
      const sliderDimensions = isVertical ? sliderHeight : sliderWidth
      const pagePosition = isVertical ? event.pageY : event.pageX
      const value = pagePosition / sliderDimensions

      setCarPosition(event.target, value, isVertical)
    }
  },
  modifiers: [
    interact.modifiers.restrict({
      restriction: 'self',
      endOnly: false
    })
  ]
})

const orientation: ComputedRef<string> = computed(() => {
  return props.vertical ? 'vertical' : 'horizontal'
})

onMounted(() => {
  const car: HTMLElement | null = document.getElementById(id)

  if (!car) return

  setSize(car, props.large, props.vertical)
  setRelativePosition(car, props.relativePosition.x, props.relativePosition.y)
  setCarPosition(car, props.carPosition, props.vertical)
})
</script>
<template>
  <div
    :id="id"
    class="draggable-car"
    :data-orientation="orientation"
    :data-relative-position-x="props.relativePosition.x"
    :data-relative-position-y="props.relativePosition.y"
  >
    <div v-if="debug">
      <p>{{ id }}</p>
      <p>orientation: {{ orientation }}</p>
      <p>x: {{ props.relativePosition.x }} / y: {{ props.relativePosition.y }}</p>
    </div>
  </div>
</template>
<style scoped>
.draggable-car {
  background-color: rgba(80, 80, 80, 0.5);
  z-index: 1;
  position: absolute;
  --car-width: 2em;
  --car-height: 2em;
  /* --car-background-image: url(); */
}

/* the slider handle */
.draggable-car:before {
  cursor: grab !important;
  content: '';
  display: block;
  position: relative;

  width: var(--car-width);
  height: var(--car-height);
  border: solid 1px transparent;
  background-color: #000;

  box-sizing: border-box;
  /* background-image: var(--car-background-image); */
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
}
</style>
