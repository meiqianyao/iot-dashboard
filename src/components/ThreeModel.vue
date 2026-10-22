<template>
  <!-- 3D容器，你可以在外层控制宽高，比如width: 100%;height: 380px -->
  <div ref="container" style="width:100%;height:100%;border-radius:8px;"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

let scene, camera, renderer, controls, cube
const container = ref(null)

onMounted(() => {
  // 1. 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x111827)

  // 2. 相机
  camera = new THREE.PerspectiveCamera(
    60,
    container.value.clientWidth / container.value.clientHeight,
    0.1,
    1000
  )
  camera.position.set(3, 2, 4)

  // 3. 渲染器 WebGL
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  container.value.appendChild(renderer.domElement)

  // 4. 轨道控制器：鼠标拖拽旋转、滚轮缩放、右键平移
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true // 阻尼，顺滑效果
  controls.dampingFactor = 0.05

  // 5. 灯光（不加灯光模型是黑的！）
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)
  const dirLight = new THREE.DirectionalLight(0xffffff, 0.8)
  dirLight.position.set(5, 10, 7.5)
  scene.add(dirLight)

  // 6. 创建立方体（模拟工业设备箱体）
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshStandardMaterial({ color: 0x3b82f6 })
  cube = new THREE.Mesh(geometry, material)
  scene.add(cube)

  // 7. 网格辅助线
  const gridHelper = new THREE.GridHelper(10, 10, 0x444444, 0x222222)
  scene.add(gridHelper)

  // 8. 动画循环
  function animate() {
    requestAnimationFrame(animate)
    controls.update()
    cube.rotation.y += 0.003
    renderer.render(scene, camera)
  }
  animate()

  // 窗口大小自适应
  const resizeHandler = () => {
    // 新增空值拦截
    if (!container.value || !camera || !renderer) return
    camera.aspect = container.value.clientWidth / container.value.clientHeight
    camera.updateProjectionMatrix()
    renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  }
  window.addEventListener('resize', resizeHandler)
})

// 销毁释放资源，Vue组件销毁必须写，面试加分点
onUnmounted(() => {
  renderer.dispose()
  controls.dispose()
})
</script>
