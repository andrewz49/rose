<template>

  <div class="page-rose">
    <div class="start-page" v-if="isStartShow">
      <div class="face" ref="face">
        <div class="eye left"></div>
        <div class="eye right"></div>
        <div class="mouth"></div>
      </div>
      <button class="btn-smail" @click="makeSmail">Make A Smile</button>
    </div>
    <div class="text">
      <div>Every moment with you is a treasure.</div>
    </div>
    <div class="btn-music">
      <img v-if="isMucicOpen" @click="musicSwitch(0)" style="width:22px" src="../assets/music_fill.png" alt="">
      <img v-else @click="musicSwitch(1)" style="width:22px" src="../assets/music_forbid_fill.png" alt="">
    </div>
    <div ref="loadingDom" class="loading">
      <img class="icon" src="../assets/sakura-flower.png" alt="">
      <div>Please waiting...</div>
    </div>
    <div class="" ref="roseDom"></div>
  </div>
</template>


<script setup>
import { ref, onMounted } from "vue"
import { getUrl } from "@/util/index"
import * as THREE from 'three';
import { FBXLoader } from 'three-stdlib';

// 交互逻辑
let isInteracting = false;
let previousPosition = { x: 0, y: 0 };

const loadingDom = ref("")
const roseDom = ref("")
const isLoad = ref(false);
const isMucicOpen = ref(true)
const isStartShow = ref(true)
// 玫瑰花模型
let roseModel = null;

const audioPath = getUrl('/weddingSong.mp3');
const audioPlayer = playAudio(audioPath);


let face;
function makeSmail() {
  face.classList.toggle("smile"); // 切换笑容状态
  setTimeout(()=>{
    isStartShow.value = false

    roseDom.value.classList.toggle("fadein");
    audioPlayer.play();
  },1000)
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



onMounted(() => {
  face = document .querySelector(".face")


  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(0, 4.6, 3.6);  // 试试调整到 (0, 2, 10)
  camera.lookAt(0, 1, -1);  // 确保相机朝向场景中心

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  roseDom.value.replaceChildren(renderer.domElement);
  // 添加光源
  const light = new THREE.AmbientLight('#fff', 0.5); // 环境光
  scene.add(light);
  const directionalLight = new THREE.DirectionalLight('#fff', 1); // 平行光
  directionalLight.position.set(10, 15, 5);
  scene.add(directionalLight);

  const pointLight = new THREE.PointLight('#fff', 0.5, 50); // 点光源
  pointLight.position.set(0, 1, 3); // 放在茎部附近
  scene.add(pointLight);

  scene.background = new THREE.Color('#c9e3e7');
  const textureLoader = new THREE.TextureLoader();


  const m_petal_img = getUrl('/textures/m_petal_roughness.jpg');
  const m_stem_img = getUrl('/textures/m_stem_roughness.jpg');
  const m_leafs_img = getUrl('/textures/m_leafs_roughness.jpg');
  const m_thorns_img = getUrl('/textures/m_thorns_roughness.jpg');
  const rose_fbx = getUrl('/fbx/rose01.fbx');

  const m_petal = textureLoader.load(m_petal_img);
  const m_stem = textureLoader.load(m_stem_img);
  const m_leafs = textureLoader.load(m_leafs_img);
  const m_thorns = textureLoader.load(m_thorns_img);



  // 加载 FBX 模型
  const loader = new FBXLoader();
  loader.load(
    rose_fbx, // 模型文件路径
    (object) => {
      object.traverse((child) => {
        if (child.isMesh) {
          // 给每个 Mesh 添加纹理
          child.material.forEach(item => {
            if (item.name == 'm_petal') { // 花朵
              item.map = m_petal;
              item.needsUpdate = true
            }
            if (item.name == 'm_stem') { // 茎
              item.map = m_stem;
              item.emissive = new THREE.Color(0x355e3b);
              item.emissiveIntensity = 0.2;
              item.needsUpdate = true
            }
            if (item.name == 'm_leafs') { // 叶
              item.map = m_leafs;
              item.roughness = 0.2;
              item.metalness = 0.0;
              item.emissive = new THREE.Color(0x6b8e23);
              item.emissiveIntensity = 0.16;
              item.needsUpdate = true;
            }
            if (item.name == 'm_thorns') { // 刺
              item.map = m_thorns;
              item.needsUpdate = true
            }
          })
        }
      });
      scene.add(object);
      object.scale.set(0.034, 0.034, 0.034);  // 根据需要调整模型大小
      setInterval(
        () => {
          object.rotation.y += 0.01
          renderer.render(scene, camera);
        }, 30
      )

      setTimeout(() => {
        loadingDom.value.remove();
        isLoad.value = true;
      }, 60)

      roseModel = object

      // 添加事件监听器
      window.addEventListener('mousedown', onMouseDown);
      window.addEventListener('mousemove', onMouseMove);
      window.addEventListener('mouseup', onMouseUp);

      window.addEventListener('touchstart', onTouchStart);
      window.addEventListener('touchmove', onTouchMove);
      window.addEventListener('touchend', onTouchEnd);
    },
    (xhr) => {
      console.log((xhr.loaded / xhr.total) * 100 + '% loaded');
    },
    (error) => {
      console.error('Error loading FBX:', error);
    }
  );

})


// 处理鼠标事件
function onMouseDown(event) {
  isInteracting = true;
  previousPosition.x = event.clientX;
  previousPosition.y = event.clientY;
}

function onMouseMove(event) {
  if (isInteracting && roseModel) {
    const deltaX = event.clientX - previousPosition.x;
    const deltaY = event.clientY - previousPosition.y;

    roseModel.rotation.y += deltaX * 0.005;
    roseModel.rotation.x += deltaY * 0.005;

    // 限制垂直旋转角度
    roseModel.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, roseModel.rotation.x));

    previousPosition.x = event.clientX;
    previousPosition.y = event.clientY;
  }
}

function onMouseUp() {
  isInteracting = false;
}

// 处理触摸事件
function onTouchStart(event) {
  if (event.touches.length === 1) {
    isInteracting = true;
    previousPosition.x = event.touches[0].clientX;
    previousPosition.y = event.touches[0].clientY;
  }
}

function onTouchMove(event) {
  if (isInteracting && roseModel && event.touches.length === 1) {
    const deltaX = event.touches[0].clientX - previousPosition.x;
    const deltaY = event.touches[0].clientY - previousPosition.y;

    roseModel.rotation.y += deltaX * 0.005;
    roseModel.rotation.x += deltaY * 0.005;

    // 限制垂直旋转角度
    roseModel.rotation.x = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, roseModel.rotation.x));

    previousPosition.x = event.touches[0].clientX;
    previousPosition.y = event.touches[0].clientY;
  }
}

function onTouchEnd() {
  isInteracting = false;
}

function playAudio(audioSrc) {
  // 创建一个新的 Audio 对象
  const audio = new Audio(audioSrc);
  audio.loop = true; // 开启循环播放
  return audio; // 返回 Audio 对象以便控制
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
  font-size: 11px;
  color: #CC4C3E;
}

.page-rose .loading {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 4;
  background-color: #fff;
  font-size: 12px;
  color: pink;
}

.loading .icon {
  animation: rotate 3s linear infinite;
  transform-origin: center;

  width: 40px;
}

.fadein {
  animation: fadeIn 2s linear;
}

.btn-music {
  z-index: 2;
  position: absolute;
  right: 16px;
  top: 10px;
}

.start-page {
  width: 100%;
  height: 100vh;
  background-color: #24A8B7;
  position: absolute;
  z-index: 3;
  padding-top: 20vh;
  box-sizing: border-box;
}

.face {
  position: relative;
  width: 200px;
  height: 200px;
  background-color: #ffcc00;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
  margin: 0 auto;
  margin-bottom: 60px;
}

.eye {
  position: absolute;
  width: 20px;
  height: 20px;
  background-color: #000;
  border-radius: 50%;
}

.eye.left {
  top: 60px;
  left: 50px;
}

.eye.right {
  top: 60px;
  right: 50px;
}

.mouth {
  position: absolute;
  bottom: 50px;
  width: 100px;
  border-bottom: 6px solid #000;
  transition: all 0.8s ease;
}

/* 笑容状态 */
.smile .mouth {
  height: 60px;
  border-radius: 60%;
}

.btn-smail {

  margin-left: 50%;
  transform: translateX(-50%);

  padding: 16px 40px;
  border: none;
  color: black;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
  font-size: 16px;
  font-family: fantasy;
}

.btn-smail:hover {
  background-color: #ff5100;
  color: #fff;
}


@keyframes fadeIn {
  from {
    opacity: 0.3;
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