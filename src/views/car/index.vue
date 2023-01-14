<template>
  <div ref="statsRef"></div>
  <div class="container" ref="containerRef"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, sRGBEncoding, ACESFilmicToneMapping } from 'three'
import * as dat from 'dat.gui'

const scene = new Scene()
const containerRef = ref<HTMLDivElement>()
const statsRef = ref<HTMLDivElement>()
const cameraRef = ref<PerspectiveCamera>()
const rendererRef = ref<WebGLRenderer>()

const controlRef = ref({
    bodyColor: '#0c0c0c',
    glassColor: '#0c0c0c',
    detailsColor: '#0c0c0c',
})


const initCamera = () => {
    cameraRef.value = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
    cameraRef.value.position.set(-30, 40, 30)
    cameraRef.value.lookAt(scene.position)
}

const initRenderer = () => {
    rendererRef.value = new WebGLRenderer({antialias: true})
    rendererRef.value.setPixelRatio(window.devicePixelRatio)
    rendererRef.value.setSize(window.innerWidth, window.innerHeight)
    rendererRef.value.outputEncoding = sRGBEncoding
    rendererRef.value.toneMapping = ACESFilmicToneMapping
    rendererRef.value.toneMappingExposure = 0.85
}

const initGui = () => {
    if (document.querySelectorAll('.dg.ac > .dg.main.a').length === 0) {
    const gui = new dat.GUI()
    gui.addColor(controlRef.value, 'bodyColor')
    gui.addColor(controlRef.value, 'glassColor')
    gui.addColor(controlRef.value, 'detailsColor')
    
  }
}

initGui()
onMounted(() => {
    initCamera()
    initRenderer()
})
</script>

<style lang="scss" scoped></style>
