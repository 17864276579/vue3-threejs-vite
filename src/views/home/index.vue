<template>
  <div ref="statsRef" ></div>
  <div ref="containerRef" class="container"></div>
</template>

<script lang="ts" setup>
import {
  AxesHelper,
  BoxGeometry,
  BufferGeometry,
  Color,
  CylinderGeometry,
  Mesh,
  MeshLambertMaterial,
  MeshBasicMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  Scene,
  SphereGeometry,
  SpotLight,
  WebGLRenderer,
  OctahedronGeometry,
  TetrahedronGeometry,
  TorusGeometry,
  OrthographicCamera,
  AmbientLight,
  PointLight,
CameraHelper,
} from 'three'
import { ref, onMounted, watch } from 'vue'
import Stats from 'stats.js'
import * as dat from 'dat.gui'
import { createMultiMaterialObject } from 'three/examples/jsm/utils/SceneUtils'

const containerRef = ref<HTMLDivElement>()
const statsRef = ref<HTMLDivElement>()
const cameraRef = ref<PerspectiveCamera | OrthographicCamera>()
//交互控制
const controlRef = ref({
  rotationSpeed: 0.02,
  bouncingSpeed: 0.03,
  toggleMaterial: () => {
    if (scene.overrideMaterial) {
      scene.overrideMaterial = null
      return
    }
    scene.overrideMaterial = new MeshLambertMaterial({
      color: 0xffffff,
    })
  },
  scaleX: 1,
  scaleY: 1,
  scaleZ: 1,
  camera: 'Perspective',
  switchCamera: () => {
    if (controlRef.value.camera === 'Perspective') {
      controlRef.value.camera = 'Orthographic'
    } else {
      controlRef.value.camera = 'Perspective'
    }
  },
  ambientColor: '#0c0c0c',
  pointLightColor: '#0c0c0c',
  debug: false,
  target: 'plane'
})

const initGui = () => {
  if (document.querySelectorAll('.dg.ac > .dg.main.a').length === 0) {
    const gui = new dat.GUI()
    gui.add(controlRef.value, 'rotationSpeed', 0, 0.5)
    gui.add(controlRef.value, 'bouncingSpeed', 0, 0.5)
    gui.add(controlRef.value, 'toggleMaterial')
    gui.add(controlRef.value, 'switchCamera')
    gui.add(controlRef.value, 'camera').listen()
    gui.addColor(controlRef.value, 'ambientColor')
    gui.add(controlRef.value, 'debug')

    const scaleFolder = gui.addFolder('scale')
    scaleFolder.add(controlRef.value, 'scaleX', 0, 5)
    scaleFolder.add(controlRef.value, 'scaleY', 0, 5)
    scaleFolder.add(controlRef.value, 'scaleZ', 0, 5)
    gui.addColor(controlRef.value, 'pointLightColor')
    gui.add(controlRef.value, 'target', ['plane', 'cube', 'sphere'])
  }
}

initGui()

watch(() => controlRef.value.target, target => {
  if (target === 'cube' ) {
    spotLight_1.target = cube
  } else if (target === 'sphere') {
    spotLight_1.target = sphere
  } else {
    spotLight_1.target = plane
  }
})

//调试相机
watch(() => controlRef.value.debug, (isDebug) => {
  isDebug ? scene.add(cameraHelper) : scene.remove(cameraHelper)
})

watch(() => controlRef.value.pointLightColor, () => {
  pointLight.color = new Color(controlRef.value.pointLightColor)
})

//更改相机投影
watch(
  () => controlRef.value.camera,
  () => {
    if (cameraRef.value instanceof PerspectiveCamera) {
      cameraRef.value = new OrthographicCamera(
        window.innerWidth / -16,
        window.innerWidth / 16,
        window.innerHeight / 16,
        window.innerHeight / -16,
        1,
        1000
      )
      cameraRef.value?.position.set(-120, 60, 180)
      cameraRef.value?.lookAt(scene.position)
    } else {
      cameraRef.value = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
      cameraRef.value.position.set(-120, 60, 180)
      cameraRef.value.lookAt(scene.position)
    }
  }
)

//更改几何体属性
watch(
  () => controlRef.value.scaleX,
  newValue => {
    cube.scale.setX(newValue)
  }
)
watch(
  () => controlRef.value.scaleY,
  newValue => {
    cube.scale.setY(newValue)
  }
)
watch(
  () => controlRef.value.scaleZ,
  newValue => {
    cube.scale.setZ(newValue)
  }
)

watch(
  () => controlRef.value.ambientColor,
  newColor => {
    console.log(newColor)
    ambinentLight.color = new Color(newColor)
  }
)

//性能监控
const stats = new Stats()
stats.dom.style.top = '40px'
stats.dom.style.left = '20px'
stats.showPanel(0)
//创建场景
const scene = new Scene()

// 环境光
const ambinentLight = new AmbientLight(controlRef.value.ambientColor)
scene.add(ambinentLight)

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
const planeGeometry = new PlaneGeometry(100, 60)
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

//常见的几何体
// const geoms: Array<BufferGeometry> = []
// geoms.push(new CylinderGeometry(1, 4, 10))
// geoms.push(new BoxGeometry(2, 2, 2))
// geoms.push(new OctahedronGeometry(5))
// geoms.push(new TetrahedronGeometry(3))
// geoms.push(new TorusGeometry(3, 1, 5, 10))

// const materials = [
//   new MeshLambertMaterial({ color: Math.random() * 0xffffff, flatShading: true }),
//   new MeshBasicMaterial({ color: 0x000, wireframe: true }),
// ]

// geoms.forEach((bg, i) => {
//   const mesh = createMultiMaterialObject(bg, materials)
//   mesh.castShadow = true
//   mesh.position.x = -20 + i * 10
//   mesh.position.y = 15
//   scene.add(mesh)
// })


//点光源
const lookAtGeom = new SphereGeometry(1)
const looAtMesh = new Mesh(
  lookAtGeom,
  new MeshLambertMaterial({
    color: 0xff0000,
  })
)
const pointLight = new PointLight(controlRef.value.pointLightColor)
pointLight.distance = 100
pointLight.position.copy(looAtMesh.position)
scene.add(looAtMesh)
scene.add(pointLight)


//聚光灯
const spotLight_1 = new SpotLight(0xffEE00)
const cameraHelper = new CameraHelper(spotLight_1.shadow.camera)

spotLight_1.castShadow = true
spotLight_1.position.copy(looAtMesh.position)
scene.add(spotLight_1)

//动画

let step = 0
let ballStep = 0

const renderScene = () => {
  stats.update()
  // cube.rotation.x += controlRef.value.rotationSpeed
  // cube.rotation.y += controlRef.value.rotationSpeed
  // cube.rotation.z += controlRef.value.rotationSpeed

  ballStep += 0.01
  looAtMesh.position.set(
    20 + 50 * Math.cos(ballStep),
    20,
    0
  )
  spotLight_1.position.copy(looAtMesh.position)

  // step += controlRef.value.bouncingSpeed
  // sphere.position.x = 20 + 10 * Math.cos(step)
  // sphere.position.y = 2 + 10 * Math.sin(step)

  requestAnimationFrame(renderScene)
  if (cameraRef.value) {
    renderer.render(scene, cameraRef.value)
  }
}

renderScene()

onMounted(() => {
  //创建相机
  cameraRef.value = new PerspectiveCamera(15, window.innerWidth / window.innerHeight, 0.1, 1000)
  // 设置相机的位置
  cameraRef.value.position.set(-120, 180, 180)
  //设置相机朝向
  cameraRef.value.lookAt(scene.position)

  // statsRef.value?.append(stats.dom)
  containerRef.value?.appendChild(renderer.domElement)
  renderer.render(scene, cameraRef.value)
})

window.addEventListener(
  'resize',
  () => {
    if (cameraRef.value instanceof PerspectiveCamera) {
      cameraRef.value.aspect = window.innerWidth / window.innerHeight
      cameraRef.value.updateProjectionMatrix()
    }
    //更新相机投影矩阵
    renderer.setSize(window.innerWidth, window.innerHeight)
  },
  false
)
</script>

<style lang="scss" scoped>
.container {
  height: 100vh;
  width: 100vw;
}
</style>
