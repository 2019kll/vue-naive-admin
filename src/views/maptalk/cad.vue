<template>
  <CommonPage>
    <div id="map" class="containers"></div>
  </CommonPage>
</template>

<script setup>
import 'maptalks/dist/maptalks.css'
import * as THREE from 'three'
import * as maptalks from 'maptalks'
import { ThreeLayer } from 'maptalks.three'
import axios from 'axios'
import { request } from '@/utils'

let map
const buildings = [
  {
    type: 'FeatureCollection',
    features: [
      {
        id: 'w23278144',
        properties: {
          levels: 10,
        },
        geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [13.413977, 52.532063],
              [13.414156, 52.532003],
              [13.414062, 52.531902],
              [13.413939, 52.531944],
              [13.41393, 52.531936],
              [13.413952, 52.531913],
              [13.41391, 52.531869],
              [13.41383, 52.531897],
              [13.413878, 52.531952],
              [13.413977, 52.532063],
            ],
          ],
        },
        type: 'Feature',
      },
      {
        id: 'w23278147',
        properties: {
          levels: 2,
        },
        geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [13.414156, 52.532003],
              [13.414357, 52.531936],
              [13.414253, 52.531822],
              [13.414187, 52.531846],
              [13.414201, 52.531861],
              [13.414062, 52.531902],
              [13.414156, 52.532003],
            ],
          ],
        },
        type: 'Feature',
      },
    ],
  },
]
onMounted(() => {
  map = new maptalks.Map('map', {
    center: [117.1946813, 31.8378146],
    zoom: 17,
    pitch: 70,
    bearing: 180,
    baseLayer: new maptalks.TileLayer('tile', {
      urlTemplate: 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png',
      subdomains: ['a', 'b', 'c', 'd'],
      attribution:
        '&copy; <a href="http://osm.org">OpenStreetMap</a> contributors, &copy; <a href="https://carto.com/">CARTO</a>',
    }),
  })
  getBuildData()
})
let features = []
let threeLayer = []
async function getBuildData() {
  let features = []
  let cad = await axios.get('../lineUpdate.json')
  // let cad = await request.get('./lineUpdate.json')
  let threeLayer = new ThreeLayer('t', {
    forceRenderOnMoving: true,
    forceRenderOnRotating: true,
  })

  let cadData = cad.data
  let lineStrings = maptalks.GeoJSON.toGeometry(cadData)
  threeLayer.prepareToDraw = function (gl, scene, camera) {
    let me = this
    let light = new THREE.DirectionalLight(0xffffff)
    light.position.set(0, -10, 10).normalize()
    scene.add(light)

    const ambientLight = new THREE.AmbientLight('#FFF', 0.5)
    scene.add(ambientLight)

    var material = new THREE.MeshPhongMaterial({ color: 0x00ffff, transparent: true })

    let lines = lineStrings.map(function (lineString) {
      var line = threeLayer.toExtrudeLine(lineString, { altitude: 0, width: 0.2, height: 3 }, material)
      // var line = threeLayer.toExtrudeLine(lineString, { altitude: 0, width: 0.2, height: 3 }, getBuildingsMaterial())

      return line
    })
    threeLayer.addMesh(lines)
  }

  threeLayer.addTo(map)
}

function getBuildingsMaterial(color = 'red') {
  const texture = new THREE.TextureLoader().load('https://assets.qszone.com/images/avatar.jpg')
  texture.needsUpdate = true //使用贴图时进行更新
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping
  // texture.repeat.set(0.002, 0.002);
  texture.repeat.set(1, 1)
  var buildingMaterial = new THREE.MeshPhongMaterial({ color: 'rgb(255,255,255)', map: texture })

  return buildingMaterial
}
</script>

<style lang="scss">
.containers {
  width: 100%;
  height: 100%;
}
</style>
