<template>
  <div ref="statsRef"></div>
  <div ref="containerRef" class="container"></div>
</template>

<script lang="ts" setup>
import {
  AxesHelper,
  BoxGeometry,
  Color,
  Mesh,
  MeshLambertMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  Scene,
  SphereGeometry,
  SpotLight,
  WebGLRenderer,
} from 'three'
import { ref, onMounted } from 'vue'
import Stats from 'stats.js'
import * as dat from 'dat.gui'

const containerRef = ref<HTMLDivElement>()
const statsRef = ref<HTMLDivElement>()
//交互控制
const controlRef = ref({
  rotationSpeed: 0.02,
  bouncingSpeed: 0.03,
})

const initGui = () => {
  if (document.querySelectorAll('.dg.ac > .dg.main.a').length === 0) {
    const gui = new dat.GUI()
    gui.add(controlRef.value, 'rotationSpeed', 0, 0.5)
    gui.add(controlRef.value, 'bouncingSpeed', 0, 0.5)
  }
}

initGui()

//性能监控
const stats = new Stats()
stats.dom.style.top = '20px'
stats.dom.style.left = '20px'
stats.showPanel(0)
//创建场景
const scene = new Scene()
//创建摄像机
const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
//设置摄像机位置
camera.position.set(-30, 40, 30)
//设置摄像机朝向
camera.lookAt(scene.position)

const renderer = new WebGLRenderer()
renderer.setClearColor(new Color(0xeeeeee))
renderer.setSize(window.innerWidth, window.innerHeight)
renderer.shadowMap.enabled = true

const spotLight = new SpotLight(0xffffff)
spotLight.castShadow = true
spotLight.position.set(-40, 60, -10)
scene.add(spotLight)
//创建一个可视化的三维坐标系 @size 轴线大小
const axes = new AxesHelper(20)
scene.add(axes)
//二维平面几何体
const planeGeometry = new PlaneGeometry(60, 20)
//材质
const meshBasicMaterial = new MeshLambertMaterial({ color: 0xcccccc })
const plane = new Mesh(planeGeometry, meshBasicMaterial)
plane.receiveShadow = true
plane.rotation.x = -0.5 * Math.PI
plane.position.x = 15
scene.add(plane)

//正方体
const cubeGeometry = new BoxGeometry(4, 4, 4)
const cubeMaterial = new MeshLambertMaterial({ color: 0xffee00, wireframe: false })
const cube = new Mesh(cubeGeometry, cubeMaterial)
cube.castShadow = true
cube.position.set(2, 2, 2)
scene.add(cube)

//球体
const shpereGeometry = new SphereGeometry(4)
const SphereMaterial = new MeshLambertMaterial({ color: 0x7777ff, wireframe: false })
const sphere = new Mesh(shpereGeometry, SphereMaterial)
sphere.castShadow = true
sphere.position.x = 20
sphere.position.y = 4
sphere.position.z = 2

scene.add(sphere)

let step = 0

const renderScene = () => {
  stats.update()
  cube.rotation.x += controlRef.value.rotationSpeed
  cube.rotation.y += controlRef.value.rotationSpeed
  cube.rotation.z += controlRef.value.rotationSpeed

  step += controlRef.value.bouncingSpeed
  sphere.position.x = 20 + 10 * Math.cos(step)
  sphere.position.y = 2 + 10 * Math.sin(step)

  requestAnimationFrame(renderScene)
  renderer.render(scene, camera)
}

renderScene()

onMounted(() => {
  statsRef.value?.append(stats.dom)
  containerRef.value?.appendChild(renderer.domElement)
  renderer.render(scene, camera)
})

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight
  //更新相机投影矩阵
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}, false)
</script>

<style lang="scss" scoped>
.container {
  height: 100vh;
  width: 100vw;
}
</style>
