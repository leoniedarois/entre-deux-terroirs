<script setup lang="ts">
import * as d3 from 'd3'
import map from './assets/france.geojson?raw'
import {ref, onMounted} from 'vue'
import imgFEMME from './assets/femme.png'
import imgHOMME from './assets/homme.png'

const data = JSON.parse(map)

const container = ref(null)
let result = ref()
let inputKM = ref(null)
let selectedItems = []

const resetMap = () => {
  d3.selectAll('path').attr('fill', '#252525')
  // clear array
  selectedItems = []
}

const calcDistance = () => {
  if (selectedItems.length > 1) {

    const echelle = 1.65
    result = Math.trunc(Math.sqrt(Math.pow(selectedItems[1].x - selectedItems[0].x, 2) + Math.pow(selectedItems[1].y - selectedItems[0].y, 2)) * echelle)

    console.log(inputKM.value)
    console.log(result)
  }
}

onMounted(() => {
  const containerEl = container.value
  // Declare the chart dimensions
  const mapWidth = window.innerWidth
  const mapHeight = window.innerHeight

  const changeColor = (d) => {
    if (selectedItems.length < 2) {
      const departmentCode = d.srcElement.attributes[1].nodeValue
      d3.select(d.srcElement).attr("fill", "#C4AC8B")

      const department = {
        code: departmentCode,
        x: d.x,
        y: d.y
      }

      selectedItems.push(department)

      // improve with centroid
      svg.append("svg:image")
          .attr('x', selectedItems[0].x)
          .attr('y', selectedItems[0].y)
          .attr('width', 60)
          .attr('height', 130)
          .attr('dominant-baseline', 'central')
          .attr('xlink:href', imgFEMME)

      svg.append("svg:image")
          .attr('x', selectedItems[1].x)
          .attr('y', selectedItems[1].y)
          .attr('width', 60)
          .attr('height', 130)
          .attr('dominant-baseline', 'central')
          .attr('xlink:href', imgHOMME)
    }


    let line = svg
        .append('path')
        .attr('d', d3.line()([[selectedItems[1].x, selectedItems[1].y], [selectedItems[0].x, selectedItems[0].y]]))
        .attr('fill', 'transparent')
        .attr('stroke', '#C4AC8B')
        .attr('stroke-width', '7')

    console.log(selectedItems)
  }

  // Create the SVG container.
  const svg = d3
      .create('svg')
      .attr('viewBox', [0, 0, mapWidth, mapHeight])
      .attr('width', mapWidth)
      .attr('height', mapHeight)

  let projection = d3.geoMercator().center([-4.504071, 47.279229]).scale(3500).translate([700, 400])

  let geoGenerator = d3.geoPath().projection(projection)

  let paths = svg
      .selectAll('path')
      .data(data.features)
      .join('path')
      .attr('d', geoGenerator)
      .attr('code', (d) => {
        return d.properties.code
      })
      .attr("cursor", "pointer")
      .attr('fill', 'transparent')
      .attr('stroke', '#6A5335')
      .on("click", (d) => changeColor(d))

  // Append the SVG element.
  containerEl.append(svg.node())
})
</script>

<template>
  <div ref="container" class="container">
    <img class="map" src="./assets/map.png" alt="">

  </div>

  <div class="ui">
    <img class="logo" src="./assets/logo.png" alt="">

    <p class="rules">
      Combien de kilomètre y a-t-il entre <b>Fernand</b> et <b>Joséphine</b>&nbsp;?
    </p>

    <div class="wrapperResult">
      <div class="input">
        <input class="result" type="text" placeholder="0" v-model="inputKM">
        <span class="dist">km</span> {{inputKM}}
      </div>
      <button class="checkButton" @click="calcDistance">
        <img class="checkIcon" src="./assets/CheckFat.png" alt="">
      </button>
    </div>

  </div>

</template>

<style>
.container {
  overflow: hidden;
  position: relative;
}

.map {
  height: 95vh;
  position: absolute;
  top: 1%;
  left: 66%;
  transform: translate(-50%);
  pointer-events: none;
  z-index: -15;
}


.ui {
  position: absolute;
  top: 50%;
  left: 2rem;
  transform: translateY(-50%);
  background: #252525;
  border-radius: 24px;
  padding: 4rem 0;
  color: #64370E;
  font-size: 25px;
  border: 4px solid #64370E;
  background: radial-gradient(50% 50% at 50% 50%, rgba(230, 205, 163, 0.00) 0%, #E6CDA3 100%), rgba(0, 0, 0, 0.20);
  max-width: 25%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;

  .checkButton {
    border: 4px solid #64370E;
    background: #C3AB8A;
    border-radius: 12px;
    cursor: pointer;
    margin-left: 10px;

    .checkIcon {
      padding: 0 5px;
      width: 20px;
    }
  }

  .logo {
    position: absolute;
    top: -20%;
    left: 50%;

    transform: translate(-50%);
  }

  .rules {
    text-align: center;
    max-width: 75%;
    margin: 5rem 0;
  }
}

.wrapperResult {
  display: flex;

  .input {
    padding: 10px 15px;
    text-align: right;
    display: flex;
    font-weight: bold;
    font-size: 20px;
    border: 4px solid #64370E;
    border-radius: 12px;
    background: #C3AB8A;
    backdrop-filter: blur(6.5px);

    ::placeholder {
      color: #64370E;
      opacity: 1; /* Firefox */
    }

    .result {
      text-align: right;
      background: transparent !important;
      border-color: transparent !important;
      color: #64370E;
      font-size: 20px;
      font-weight: bold;
    }

    .dist {
      padding-left: 10px;
      color: #64370E;
    }
  }

}

</style>
