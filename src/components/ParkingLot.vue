<script lang="ts" setup>
import DraggableCar from './DraggableCar.vue'

const randomId = () => Math.random().toString(36).substring(7)
</script>
<template>
  <div id="parking-lot">
    <div id="car-container">
      <DraggableCar
        :id="randomId()"
        :relative-position="{ x: 0, y: 380 }"
        :car-position="0"
        :winning-car="true"
      />
      <DraggableCar :id="randomId()" :relative-position="{ x: 0, y: 650 }" :car-position="0.64" />
      <DraggableCar
        :id="randomId()"
        :relative-position="{ x: 0, y: 105 }"
        :car-position="0"
        large
      />
      <DraggableCar
        :id="randomId()"
        :relative-position="{ x: 250, y: 0 }"
        :car-position="0.55"
        vertical
      />
      <DraggableCar
        :id="randomId()"
        :relative-position="{ x: 450, y: 0 }"
        :car-position="0.73"
        vertical
      />
      <DraggableCar :id="randomId()" :relative-position="{ x: 715, y: 0 }" vertical large />
    </div>
  </div>
</template>
<style scoped>
#car-container {
  position: relative;
  width: 100%;
  height: 100%;
}

#parking-lot {
  /* border: 1px solid #ff0; */
  width: 875px;
  height: 875px;
  background-color: #010101;
  margin: 5px;
  z-index: 10;

  --b: 5px; /* thickness of the border */
  --c: yellow; /* color of the border */
  --w: 38%; /* width of border */

  border: var(--b) solid transparent;
  --_g: #0000 90deg, var(--c) 0;
  --_p: var(--w) var(--w) border-box no-repeat;
  background:
    conic-gradient(from 90deg at top var(--b) left var(--b), var(--_g)) 0 0 / var(--_p),
    conic-gradient(from 180deg at top var(--b) right var(--b), var(--_g)) 100% 0 / var(--_p),
    conic-gradient(from 0deg at bottom var(--b) left var(--b), var(--_g)) 0 100% / var(--_p),
    conic-gradient(from -90deg at bottom var(--b) right var(--b), var(--_g)) 100% 100% / var(--_p);
}
</style>
