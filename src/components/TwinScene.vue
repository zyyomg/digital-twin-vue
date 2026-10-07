<template>
  <div ref="canvasContainer" class="scene-container"></div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js';
import mqtt from 'mqtt';

const canvasContainer = ref(null);
let scene, camera, renderer, controls, client;
const deviceMap = {};

function loadModel(name, path, xPosition) {
  const loader = new GLTFLoader();
  loader.setMeshoptDecoder(MeshoptDecoder);
  loader.load(path, (gltf) => {
    const model = gltf.scene;
    model.name = name;
    
    const box = new THREE.Box3().setFromObject(model);
    const size = box.getSize(new THREE.Vector3());
    const maxDim = Math.max(size.x, size.y, size.z);
    const scale = 2 / maxDim; 
    model.scale.set(scale, scale, scale);
    
    const center = box.getCenter(new THREE.Vector3());
    model.position.x = xPosition - center.x * scale;
    model.position.y = -center.y * scale; 
    model.position.z = -center.z * scale;

    model.traverse((child) => {
      if (child.isMesh) {
        child.material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
        child.userData.deviceId = name;
      }
    });

    scene.add(model);
    deviceMap[name] = model;
    console.log(`${name} 模型加载成功！`);
  }, undefined, (error) => {
    console.error(`加载 ${path} 失败`, error);
  });
}

function updateDeviceVisual(data) {
  // 适配之前 Python 里的 device_id 字段
  const targetModel = deviceMap[data.device_id]; 
  if (!targetModel) return;

  let color = 0x00ff00;
  const temp = data.temperature;
  if (temp > 80) color = 0xff0000;
  else if (temp > 60) color = 0xffff00;

  targetModel.traverse((child) => {
    if (child.isMesh) child.material.color.setHex(color);
  });
}

onMounted(() => {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xdddddd);
  
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(0, 3, 8);

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  canvasContainer.value.appendChild(renderer.domElement);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  scene.add(new THREE.AmbientLight(0xffffff, 1.5));
  const dirLight = new THREE.DirectionalLight(0xffffff, 2);
  dirLight.position.set(10, 20, 10);
  scene.add(dirLight);

  loadModel('motor_01', '/models/motor.glb', -4);
  loadModel('pump_01', '/models/pump.glb', 0);
  loadModel('fan_01', '/models/fan.glb', 4);

  // 真实虚拟机IP
  const MQTT_BROKER_IP = '192.168.94.130'; 
  client = mqtt.connect(`ws://${MQTT_BROKER_IP}:9001`);
  
  client.on('connect', () => {
    console.log('成功连接到 MQTT Broker!');
    client.subscribe('factory/line1/+/data', (err) => {
      if (!err) console.log('已订阅设备数据主题');
    });
  });

  client.on('message', (topic, message) => {
    try {
      const data = JSON.parse(message.toString());
      updateDeviceVisual(data);
    } catch (e) {}
  });

  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  }
  animate();

  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  if (client) client.end();
  if (renderer) renderer.dispose();
});

const handleResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
};
</script>

<style scoped>
.scene-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>