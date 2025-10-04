<template>
  <div ref="mountRef" class="scene-container"></div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from "vue";
import * as THREE from "three";

defineOptions({
  ssr: false,
});

const props = defineProps<{
  width: number;
  height: number;
  depth: number;
  rows: number;
  cols: number;
  color: string;
}>();

const mountRef = ref<HTMLDivElement | null>(null);

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let shelf: THREE.Group;

function createShelf() {
  if (shelf) scene.remove(shelf);

  const group = new THREE.Group();

  // Корпус
  const boxGeo = new THREE.BoxGeometry(props.width, props.height, props.depth);
  const boxMat = new THREE.MeshStandardMaterial({ color: props.color, metalness: 0.1, roughness: 0.7 });
  const box = new THREE.Mesh(boxGeo, boxMat);
  group.add(box);

  // Горизонтальные полки
  const shelfThickness = 0.02;
  for (let i = 1; i < props.rows; i++) {
    const y = -props.height / 2 + (i * props.height) / props.rows;
    const shelfGeo = new THREE.BoxGeometry(props.width, shelfThickness, props.depth);
    const shelfMat = new THREE.MeshStandardMaterial({ color: "#e0e0e0" });
    const shelf = new THREE.Mesh(shelfGeo, shelfMat);
    shelf.position.y = y;
    group.add(shelf);
  }

  // Вертикальные перегородки
  for (let j = 1; j < props.cols; j++) {
    const x = -props.width / 2 + (j * props.width) / props.cols;
    const dividerGeo = new THREE.BoxGeometry(shelfThickness, props.height, props.depth);
    const dividerMat = new THREE.MeshStandardMaterial({ color: "#e0e0e0" });
    const divider = new THREE.Mesh(dividerGeo, dividerMat);
    divider.position.x = x;
    group.add(divider);
  }

  shelf = group;
  scene.add(shelf);
}

onMounted(() => {
  if (!mountRef.value) return;

  // Сцена
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xf5f5f5);

  // Камера
  camera = new THREE.PerspectiveCamera(60, mountRef.value.clientWidth / mountRef.value.clientHeight, 0.1, 1000);
  camera.position.set(2, 1.5, 3);
  camera.lookAt(0, 0.7, 0);

  // Рендерер
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(mountRef.value.clientWidth, mountRef.value.clientHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  mountRef.value.appendChild(renderer.domElement);

  // Освещение
  const ambientLight = new THREE.AmbientLight(0xffffff, 1.2);
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(5, 5, 5);
  scene.add(directionalLight);

  // Пол
  const floorGeometry = new THREE.PlaneGeometry(10, 10);
  const floorMaterial = new THREE.MeshStandardMaterial({ color: 0x8b4513, roughness: 0.9 });
  const floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI / 2;
  scene.add(floor);

  // Человек (упрощённый силуэт)
  const humanGeometry = new THREE.BoxGeometry(0.4, 1.7, 0.25);
  const humanMaterial = new THREE.MeshBasicMaterial({ color: 0x4a90e2, transparent: true, opacity: 0.4 });
  const human = new THREE.Mesh(humanGeometry, humanMaterial);
  human.position.set(-1.8, 0.85, 0);
  scene.add(human);

  // Функция создания шкафа

  createShelf();

  // Анимация
  const animate = () => {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  };
  animate();

  // Обработка resize
  const handleResize = () => {
    if (!mountRef.value) return;
    camera.aspect = mountRef.value.clientWidth / mountRef.value.clientHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(mountRef.value.clientWidth, mountRef.value.clientHeight);
  };
  window.addEventListener("resize", handleResize);

  // Cleanup
  onUnmounted(() => {
    window.removeEventListener("resize", handleResize);
    if (mountRef.value && renderer.domElement) {
      mountRef.value.removeChild(renderer.domElement);
    }
  });
});

// Пересоздание шкафа при изменении пропсов
watch(
  () => [props.width, props.height, props.depth, props.rows, props.cols, props.color],
  () => {
    if (scene && camera && renderer) {
      createShelf();
    }
  }
);
</script>

<style scoped>
.scene-container {
  width: 100vw;
  height: 100vh; /* или min-height: 500px */
  background: #000; /* для отладки */
}
</style>
