<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { Water } from "three/examples/jsm/objects/Water2";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2000);
camera.position.set(-50, 50, 130);
camera.updateProjectionMatrix();
const renderer = new THREE.WebGLRenderer({
  antialias: true,

  //对数深度缓冲区，解决小岛上多个面闪烁（黑色）的问题
  logarithmicDepthBuffer: true,
});
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.setSize(window.innerWidth, window.innerHeight);

//添加球体：
const skyGeometry = new THREE.SphereGeometry(1000, 60, 60);
const skyMaterial = new THREE.MeshBasicMaterial({
  map: new THREE.TextureLoader().load("../public/textures/sky.jpg"),
});

const sphere = new THREE.Mesh(skyGeometry, skyMaterial);
sphere.geometry.scale(1, 1, -1);
scene.add(sphere);

//视频纹理：
const video = document.createElement("video");
video.muted = "muted";
video.src = "../public/textures/sky.mp4";
video.loop = true;

const container = ref(null);
const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

window.addEventListener("click", (e) => {
  if (video.paused) {
    video.play();
    skyMaterial.map = new THREE.VideoTexture(video);
    skyMaterial.map.needsUpdate = true;
  }
});

window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

//添加水面：
const waterGeometry = new THREE.CircleGeometry(300, 64);
const water = new Water(waterGeometry, {
  textureWidth: 1024,
  textureHeight: 1024,
  color: 0xeeeeff,
  flowDirection: new THREE.Vector2(1, 1),
  scale: 1,
});
water.position.y = -49;

//载入环境纹理：
const hdrLoader = new RGBELoader();
hdrLoader.loadAsync("../public/textures/050.hdr").then((texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
});

//添加平行光：
const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(-100, 100, 10);
scene.add(directionalLight);

//添加小岛：
const gltfLoader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("../public/draco/");
gltfLoader.setDRACOLoader(dracoLoader);
gltfLoader.load("../public/model/island2.glb", (gltf) => {
  const island = gltf.scene;
  island.position.y = -50;
  scene.add(gltf.scene);
});

water.rotation.x = -Math.PI / 2;
scene.add(water);

onMounted(() => {
  const controls = new OrbitControls(camera, container.value);
  controls.enableDamping = true;
  container.value.appendChild(renderer.domElement);
  render();
});
</script>

<template>
  <div class="container" ref="container"></div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
.container {
  height: 100vh;
  width: 100vw;
  background-color: #1e1a20;
}
</style>
