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
const getElement = (): HTMLElement => {
	const el = document.getElementById(id)

	if (!el) {
		throw new Error(`Element with id ${id} not found`)
	}

	return el
}
const debug: Ref<Boolean> = ref(true)

const setRelativePosition = (x: number, y: number): void => {
  getElement().style.transform = `translate(${x}px, ${y}px)`
}

const setSize = (large: Boolean, vertical: Boolean): void => {
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

  getElement().style.width = containerWidth
  getElement().style.height = containerHeight
  getElement().style.setProperty('--car-width', carWidth)
  getElement().style.setProperty('--car-height', carHeight)
}

const setCarPosition = (value: number, isVertical: boolean) => {
  if (value >= 0.2) {
    if (isVertical) {
      const carHeight = parseFloat(getElement().style.getPropertyValue('--car-height'))
      const carHeightPercentage = carHeight / interact.getElementRect(getElement()).height
      const valueWithCarHeight = value - carHeightPercentage

      getElement().style.paddingTop = valueWithCarHeight * 100 + '%'
    } else {
      const carWidth = parseFloat(getElement().style.getPropertyValue('--car-width'))
      const carWidthPercentage = carWidth / interact.getElementRect(getElement()).width
      const valueWithCarWidth = value - carWidthPercentage

      getElement().style.paddingLeft = valueWithCarWidth * 100 + '%'
    }
  } else {
    getElement().style.paddingTop = '0'
    getElement().style.paddingLeft = '0'
  }
  getElement().setAttribute('data-value', value.toFixed(2))
}

interact('#' + id).draggable({
  origin: 'self',
  inertia: true,
  listeners: {
    move(event) {
      const isVertical = orientation.value === 'vertical'
      const sliderWidth = interact.getElementRect(event.target).width
      const sliderHeight = interact.getElementRect(event.target).height
      const sliderDimensions = isVertical ? sliderHeight : sliderWidth
      const pagePosition = isVertical ? event.pageY : event.pageX
      const value = pagePosition / sliderDimensions

      setCarPosition(value, isVertical)
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

const currentValue: ComputedRef<number> = computed(() => {
	try {
		return parseFloat(getElement().getAttribute('data-value') || '0')
	} catch (err) {
		return 0
	}
})

onMounted(() => {
  setSize(props.large, props.vertical)
  setRelativePosition(props.relativePosition.x, props.relativePosition.y)
  setCarPosition(props.carPosition, props.vertical)
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
      <p>value: {{ currentValue }}</p>
      <p>x: {{ props.relativePosition.x }} / y: {{ props.relativePosition.y }}</p>
    </div>
  </div>
</template>
<style scoped>
.draggable-car {
  /* background-color: rgba(80, 80, 80, 0.5); */
	background-color: transparent;
  z-index: 1;
  position: absolute;
  --car-width: 2em;
  --car-height: 2em;
	--background-color: #000;
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
  background-color: var(--background-color);

  box-sizing: border-box;
  /* background-image: var(--car-background-image); */
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
}
</style>
