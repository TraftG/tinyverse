<template>
  <div ref="sceneContainer" class="scene-container"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';

const sceneContainer = ref(null);

onMounted(() => {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    75, 
    window.innerWidth / window.innerHeight, 
    0.1, 
    1000
  );

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  sceneContainer.value.appendChild(renderer.domElement);

  const galaxyRings = [];

  const numRings = 6; 
  const numStarsPerRing = 2000; 
  const radiusStep = 50; 
  const initialRadius = 120; 

  const colors = [0xffffff, 0xffd700, 0xffd700, 0xffd700, 0x1e90ff];

  for (let j = 0; j < numRings; j++) {
    const radiusInner = initialRadius + j * radiusStep;
    const radiusOuter = radiusInner + radiusStep;

    const galaxyRing = new THREE.Group();

    for (let i = 0; i < numStarsPerRing; i++) {
      const angle = Math.random() * Math.PI * 2;
      const radius = Math.random() * (radiusOuter - radiusInner) + radiusInner;
      const x = Math.cos(angle) * radius;
      const y = Math.sin(angle) * radius;
      const z = (Math.random() - 0.5) * 50;

      const starGeometry = new THREE.SphereGeometry(1, 8, 8); // Сфера вместо точек
      const starMaterial = new THREE.MeshBasicMaterial({ 
        color: colors[Math.floor(Math.random() * colors.length)] 
      });
      const star = new THREE.Mesh(starGeometry, starMaterial);
      star.position.set(x, y, z);
      
      galaxyRing.add(star);
    }

    galaxyRing.rotation.x = Math.PI / 2;
    scene.add(galaxyRing);
    galaxyRings.push(galaxyRing);
  }

  // Фоновые звезды
  const backgroundStars = new THREE.BufferGeometry();
  const backgroundVertices = [];
  const backgroundColors = [];

  const numBackgroundStars = 5000;

  for (let i = 0; i < numBackgroundStars; i++) {
    const x = (Math.random() - 0.5) * 5000;
    const y = (Math.random() - 0.5) * 5000;
    const z = (Math.random() - 0.5) * 5000;
    backgroundVertices.push(x, y, z);

    const color = new THREE.Color(0xffffff);
    backgroundColors.push(color.r, color.g, color.b);
  }

  backgroundStars.setAttribute(
    'position',
    new THREE.Float32BufferAttribute(backgroundVertices, 3)
  );
  backgroundStars.setAttribute(
    'color',
    new THREE.Float32BufferAttribute(backgroundColors, 3)
  );

  const backgroundMaterial = new THREE.PointsMaterial({ size: 1, vertexColors: true });
  const backgroundStarField = new THREE.Points(backgroundStars, backgroundMaterial);
  scene.add(backgroundStarField);

  camera.position.z = 800;

  const animate = () => {
    requestAnimationFrame(animate);
    galaxyRings.forEach(ring => {
      ring.rotation.y += 0.002;
    });
    renderer.render(scene, camera);
  };

  animate();

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });

  let isDragging = false;
  let previousMousePosition = { x: 0, y: 0 };

  renderer.domElement.addEventListener('mousedown', () => {
    isDragging = true;
  });

  renderer.domElement.addEventListener('mouseup', () => {
    isDragging = false;
  });

  renderer.domElement.addEventListener('mousemove', (event) => {
    if (isDragging) {
      const deltaMove = {
        x: event.offsetX - previousMousePosition.x,
        y: event.offsetY - previousMousePosition.y
      };

      galaxyRings.forEach(ring => {
        ring.rotation.y += deltaMove.x * 0.005;
        ring.rotation.x += deltaMove.y * 0.005;
      });
    }
    previousMousePosition = {
      x: event.offsetX,
      y: event.offsetY
    };
  });

  renderer.domElement.addEventListener('wheel', (event) => {
    camera.position.z += event.deltaY * 0.5;
    camera.position.z = Math.max(300, Math.min(1500, camera.position.z));
  });
});
</script>

<style>
.scene-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>

<script>
export default {
  name: 'GalaxyRingScene',
};
</script>
