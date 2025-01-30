<template>

  <div class="page-rose">
    <div class="text">
      <div>Every moment with you is a treasure.</div>
    </div>
    <h3 style="width: 100vw;height: 40px;color: white;text-align: center;font-weight: normal;font-family: cursive;position: fixed;top: 0;">祝亲爱的余恩利妈妈新年快乐</h3>
    <div class="btn-music">
      <img class="icon" ref="musicDom" v-if="isMucicOpen" @click="musicSwitch(0)"  src="../assets/music_fill.svg" >
      <img class="icon" v-else @click="musicSwitch(1)"  src="../assets/music_forbid_fill.svg" >
      
    </div>
    <div ref="loadingDom" class="loading">
      <div class="progress-bar">
        <div id="progress-bar-inner" class="progress-bar-inner">
          <div id="progress-text" class="progress-text">0%</div>
          <img class="icon" src="../assets/rose.svg">
        </div>
      </div>
    </div>
    <canvas class="roseDom" ref="roseDom"></canvas>
  </div>
</template>


<script setup>
import { ref, onMounted } from "vue"
import { getUrl,listenResize } from "@/util/index"
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import * as dat from 'lil-gui'



// Audio
const audioPath = getUrl('/weddingSong.mp3');
const audioPlayer = playAudio(audioPath);
const isMucicOpen = ref(false)

const roseDom = ref("");
const loadingDom = ref("");
const musicDom = ref("")
onMounted(() => {
  const canvas = roseDom.value
  // Scene
  const scene = new THREE.Scene()
  // Gui
  const gui = new dat.GUI()
  gui.close()
  gui.hide()
  const debugObject = {
    envMapIntensity: 1.5,
  }
  // Size
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight,
  }
  // Camera
  const camera = new THREE.PerspectiveCamera(40, sizes.width / sizes.height, 0.1, 100)
  camera.position.set(5, 3.8, 0)

  // Controls
  const controls = new OrbitControls(camera, canvas)
  controls.enableDamping = true
  controls.zoomSpeed = 0.3

  const loadingManager = new THREE.LoadingManager()

  loadingManager.onStart = (url, itemsLoaded, itemsTotal) => {
    console.log(`Started loading file: ${url}.\nLoaded ${itemsLoaded} of ${itemsTotal} files.`)
  }

  
  loadingManager.onLoad = () => {
    console.log('Loading complete!')
    console.log('success')
    setTimeout(() => {
      
      roseDom.value.classList.toggle("fadein");
      setTimeout(() => {        
        loadingDom.value.remove();
      }, 200)
    }, 200)
  }

  const progressEle = document.querySelector('#progress-bar-inner')
  const progressText = document.querySelector('#progress-text')
  loadingManager.onProgress = (url, itemsLoaded, itemsTotal) => {
    console.log(`Loading file: ${url}.\nLoaded ${itemsLoaded} of ${itemsTotal} files.`)
    // progress-bar-inner
    // transform 0% -> 100%
    if (itemsTotal > 2) {

      progressEle.style.width = `${(itemsLoaded / itemsTotal) * 100}%`
      progressText.innerHTML = `${(itemsLoaded / itemsTotal).toFixed(2)  * 100}%`
    }
  }
  loadingManager.onError = (url) => {
    console.log(`There was an error loading ${url}`)
  }

  /**
   * Loaders
   */
  const gltfLoader = new GLTFLoader(loadingManager)

  /**
   * Update all materials
   */
  const updateAllMaterials = () => {
    scene.traverse((child) => {
      if (child instanceof THREE.Mesh && child.material instanceof THREE.MeshStandardMaterial) {
        console.log(child)
        child.material.envMapIntensity = debugObject.envMapIntensity
        child.castShadow = true
        child.receiveShadow = true
      }
    })
  }

  let model;
  /**
   * Models
   */
  gltfLoader.load(
    "https://image.andrewhome.club/scene.gltf",
    (gltf) => {
      gltf.scene.scale.set(3.5, 3.5, 3.5)
      gltf.scene.position.set(0, -2.5, 0)
      gltf.scene.rotation.set(0, Math.PI * 0.5, 0)
      model = gltf.scene
      setTimeout(() => {
        scene.add(gltf.scene)
      }, 200)
      updateAllMaterials()
    },
    (progress) => {
      console.log('progress')
      console.log(progress)
    },
  )
  
  /**
   * Light
   */
  const directionLight = new THREE.DirectionalLight('#FFF9C4', 10)
  directionLight.position.set(0.25, 3, -2.25)
  scene.add(directionLight)

  directionLight.castShadow = true

  const directionalLightCameraHelper = new THREE.CameraHelper(directionLight.shadow.camera)
  scene.add(directionalLightCameraHelper)
  directionalLightCameraHelper.visible = false

  directionLight.shadow.camera.far = 15
  directionLight.shadow.mapSize.set(1024, 1024)

  directionLight.shadow.normalBias = 0.05;

  /** axesHelper */
  const axesHelper = new THREE.AxesHelper(5)
  scene.add(axesHelper)
  axesHelper.visible = false

  const ambientLight = new THREE.AmbientLight('#FFF9C4', 1)
  scene.add(ambientLight)

  // Renderer
  const renderer = new THREE.WebGLRenderer({
    canvas,
    antialias: true,
    alpha: true,
  })
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.physicallyCorrectLights = true
  renderer.outputEncoding = THREE.sRGBEncoding
  renderer.toneMapping = THREE.ReinhardToneMapping
  renderer.toneMappingExposure = 2.5
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap

  // rotation control
  const rotation = {
    autoRotation: true,
  }
  gui.add(rotation, 'autoRotation')

  // Animations
  const tick = () => {
    // stats.begin()
    controls.update()
    if (model && rotation.autoRotation) {
      model.rotation.y += 0.005
    }
    // Render
    renderer.render(scene, camera)
    // stats.end()
    requestAnimationFrame(tick)
  }
  tick()

  listenResize(sizes, camera, renderer)
})

// 创建一个新的 Audio 对象
function playAudio(audioSrc) {
  const audio = new Audio(audioSrc);
  audio.loop = true; // 开启循环播放
  return audio; // 返回 Audio 对象以便控制
}
function musicSwitch(status) {
  if (status) {
    audioPlayer.play();
    isMucicOpen.value = true
  } else {
    audioPlayer.pause();
    isMucicOpen.value = false
  }
}



</script>


<style scoped>
.page-rose {
  width: 100%;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.page-rose .text {
  z-index: 2;
  position: absolute;
  right: 10px;
  bottom: 10px;
  font-size: 14px;
  color: #f48160;
  font-family:Verdana, Geneva, Tahoma, sans-serif;
}

.page-rose .loading {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;

  box-sizing: border-box;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 4;
  background-color: #004D40;
  font-size: 12px;
  color: pink;
}
.loading .progress-bar{
  width: 80vw;
  height: 16px;
  border-radius: 12px;
  background-color: rgb(167, 167, 167);
 
  display: flex;
  align-items: center;
  margin: 0 auto;
}
.loading .progress-bar-inner{
  position: relative;
  width: 0%;
  height: 16px;
  border-radius: 12px;
  background-color: #e0bf75;
  display: flex;
  align-items: center;
}
.loading .progress-bar .icon{
  position: absolute;
  right: -8px;
  width: 22px;
  margin-top: 2px;
  color: white;
} 

.loading .progress-text {
  color: white;

  text-indent: 16px;
}


.fadein {
  animation: fadeIn 2s linear;
}
.rotate {
  animation: rotate 2s linear;
}
.btn-music {
  z-index: 2;
  position: absolute;
  right: 20px;
  top: 14px;
  
}
.btn-music img{
  width: 28px;
  color: white;
}




@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(365deg);

  }
}
</style>