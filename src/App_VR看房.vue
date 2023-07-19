<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 0.1;

const renderer = new THREE.WebGL1Renderer();
renderer.setSize(window.innerWidth, window.innerHeight);

// const arr = ["4_l", "4_r", "4_u", "4_d", "4_b", "4_f"];
// const boxMaterial = [];
// arr.forEach((item) => {
//   let texture = new THREE.TextureLoader().load(`../public/imgs/living/${item}.jpg`);
//   if (item === "4_u" || item === "4_d") {
//     texture.rotation = Math.PI;
//     texture.center = new THREE.Vector2(0.5, 0.5);
//     boxMaterial.push(new THREE.MeshBasicMaterial({ map: texture }));
//   } else {
//     boxMaterial.push(new THREE.MeshBasicMaterial({ map: texture }));
//   }
// });

// //添加立方体：
// const geometry = new THREE.BoxGeometry(10, 10, 10);
// const cube = new THREE.Mesh(geometry, boxMaterial);
// cube.geometry.scale(1, 1, -1);

//添加球体：
const geometry = new THREE.SphereGeometry(5, 32, 32);
// geometry.scale(1, 1, -1);
const loader = new RGBELoader();
loader.load("../public/imgs/hdr/Living.hdr", (texture) => {
  const material = new THREE.MeshBasicMaterial({ map: texture });
  const sphere = new THREE.Mesh(geometry, material);
  sphere.geometry.scale(1, 1, -1);
  scene.add(sphere);
});

const container = ref(null);
const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

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
  background-color: #f0f0f0;
}
</style>
