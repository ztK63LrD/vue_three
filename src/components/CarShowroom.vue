<script setup>
import * as THREE from "three"
import { ref, reactive } from 'vue'
// 添加轨道控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
// 引入gui.js库
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
// 引入tween.js库
import * as TWEEN from '@tweenjs/tween.js'
// 实例化一个gui对象
const gui = new GUI()

// 创建场景
const scene = new THREE.Scene()
// 创建相机
const camera = new THREE.PerspectiveCamera(40,window.innerWidth / window.innerHeight,0.1,1000)
camera.position.set(4.25,1.4,-4.5)
// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true })
renderer.setSize(window.innerWidth,window.innerHeight)
renderer.shadowMap.enabled = true // 接收阴影
document.body.appendChild(renderer.domElement)

// 添加控制器
const controls = new OrbitControls(camera,renderer.domElement)
controls.enableDamping = true // 设置控制阻尼
controls.maxDistance = 10 // 最大缩放距离
controls.minDistance = 1 // 最小缩放距离
controls.minPolarAngle = 0 // 最小旋转角度
controls.maxPolarAngle = 85 / 360 * 2 * Math.PI // 最大旋转角度

// 监听页面变化
window.addEventListener("resize",()=>{ 
  renderer.setSize(window.innerWidth,window.innerHeight)
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
})

// 车身材质
let bodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 'red',
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1.0,
  clearcoatRoughness: 0.03
})
// 玻璃材质
let glassMaterial = new THREE.MeshPhysicalMaterial({
  color: '#793e3e',
  metalness: 0.25,
  roughness: 0,
  transmission: 1.0 // 透光性
})

// 设置门的集合
let doors = []
// 加载汽车模型
const loader = new GLTFLoader()
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath("/draco/")
loader.setDRACOLoader(dracoLoader)
loader.load("/public/model/Lamborghini.glb",(gltf)=>{
  const carModel = gltf.scene
  carModel.rotation.y = Math.PI
  carModel.traverse((obj)=>{
    if(obj.name === 'Object_103' || obj.name === 'Object_64' || obj.name === 'Object_77'){
      // 车身
      obj.material = bodyMaterial
    }else if(obj.name === 'Object_90'){
      // 玻璃
      obj.material = glassMaterial
    }else if(obj.name === 'Empty001_16' || obj.name === 'Empty002_20'){
      // 门
      doors.push(obj)
    }else{
      return true
    }
    // 车模型阴影
    obj.castShadow = true
  })
  scene.add(gltf.scene)
})

// 设置环境光源
const ambientLight = new THREE.AmbientLight('#fff',0.5)
scene.add(ambientLight)

// 设置地板样式
const floorGeometry = new THREE.PlaneGeometry(20,20)
const floormaterial = new THREE.MeshPhysicalMaterial({
  side: THREE.DoubleSide,
  color: 0x808080,
  metalness: 0, // 设置金属度
  roughness: 0.1, // 设置粗糙度
  wireframe: false // 关闭网格线
})
const mesh = new THREE.Mesh(floorGeometry,floormaterial)
mesh.rotation.x = Math.PI / 2
// mesh.receiveShadow = true // 接收阴影
scene.add(mesh)

// 设置圆柱体模拟展厅
const cylinder = new THREE.CylinderGeometry(12,12,20,32)
const cylindermaterial = new THREE.MeshPhysicalMaterial({
  color: 0x6c6c6c,
  side: THREE.DoubleSide
})
const cylinderMesh = new THREE.Mesh(cylinder,cylindermaterial)
scene.add(cylinderMesh)

// 设置聚光灯(让汽车更具有立体金属感)
const spotLight = new THREE.SpotLight('#fff',2)
spotLight.angle = Math.PI / 8 // 散射角度，和水平线的夹角
spotLight.penumbra = 0.2 // 横向，聚光锥的半影衰减百分比
spotLight.decay = 2 // 纵向，沿着光照距离的衰减量
spotLight.distance = 30
spotLight.shadow.radius = 10
spotLight.shadow.mapSize.set(4096,4096)
spotLight.position.set(-5,10,1)
spotLight.target.position.set(0,0,0) // 光照射的方向
spotLight.castShadow = true
scene.add(spotLight)

// 设置gui模板控制
// 修改默认面板名称
gui.domElement.parentNode.querySelector('.title').textContent = '3D汽车动态操作'

const bodyChange = gui.addFolder("车身材质设置")
bodyChange.close() // 默认关闭状态
bodyChange.addColor(bodyMaterial,'color').name('车身颜色').onChange(value=>{
  bodyMaterial.color.set(value)
})
bodyChange.add(bodyMaterial,'metalness',0,1).name('金属度').onChange(value=>{
  bodyMaterial.metalness = value
})
bodyChange.add(bodyMaterial,'roughness',0,1).name('粗糙度').onChange(value=>{
  bodyMaterial.roughness = value
})
bodyChange.add(bodyMaterial,'clearcoat',0,1).name('清漆强度').onChange(value=>{
  bodyMaterial.clearcoat = value
})
bodyChange.add(bodyMaterial,'clearcoatRoughness',0,1).name('清漆层粗糙度').onChange(value=>{
  bodyMaterial.clearcoatRoughness = value
})
const glassChange = gui.addFolder("玻璃设置")
glassChange.close() // 默认关闭状态
glassChange.addColor(glassMaterial,'color').name('玻璃颜色').onChange(value=>{
  glassMaterial.color.set(value)
})
glassChange.add(glassMaterial,'metalness',0,1).name('金属度').onChange(value=>{
  glassMaterial.metalness = value
})
glassChange.add(glassMaterial,'roughness',0,1).name('粗糙度').onChange(value=>{
  glassMaterial.roughness = value
})
glassChange.add(glassMaterial,'transmission',0,1).name('透光性').onChange(value=>{
  glassMaterial.transmission = value
})

// 设置汽车状态
let carStatus;
// 打开左车门
const carLeftOpen = () => { 
  carStatus = 'open'
  setAnimationDoor({ x: 0 }, { x: Math.PI / 3 }, doors[1])
}
// 打开右车门
const carRightOpen = () => { 
  carStatus = 'open'
  setAnimationDoor({ x: 0 }, { x: Math.PI / 3 }, doors[0])
}
// 关闭左车门
const carLeftClose = () => { 
  carStatus = 'close'
  setAnimationDoor({ x: Math.PI / 3 }, { x: 0 }, doors[1])
}
// 关闭右车门
const carRightClose = () => { 
  carStatus = 'close'
  setAnimationDoor({ x: Math.PI / 3 }, { x: 0 }, doors[0])
}

// 车内视角
const carIn = () => {
  setAnimationCamera({ cx: 4.25, cy: 1.4, cz: -4.5, ox: 0, oy: 0.5, oz: 0 }, { cx: -0.27, cy: 0.83, cz: 0.60, ox: 0, oy: 0.5, oz: -3 });
}
// 车外视角
const carOut = () => {
  setAnimationCamera({ cx: -0.27, cy: 0.83, cz: 0.6, ox: 0, oy: 0.5, oz: -3 }, { cx: 4.25, cy: 1.4, cz: -4.5, ox: 0, oy: 0.5, oz: 0 });
}

var obj = { carRightOpen,carLeftOpen,carRightClose,carLeftClose,carIn,carOut }
// 设置车身动态操作
const doChange = gui.addFolder("车身动态操作设置")
doChange.close() // 默认关闭状态
doChange.add(obj, "carLeftOpen").name('打开左车门')
doChange.add(obj, "carRightOpen").name('打开右车门')
doChange.add(obj, "carLeftClose").name('关闭左车门')
doChange.add(obj, "carRightClose").name('关闭右车门')
doChange.add(obj, "carIn").name('车内视角')
doChange.add(obj, "carOut").name('车外视角')

// 设置补间动画
const setAnimationDoor = (start, end, mesh) => {
  const tween = new TWEEN.Tween(start).to(end, 1000).easing(TWEEN.Easing.Quadratic.Out)
  tween.onUpdate((that) => {
    mesh.rotation.x = that.x
  })
  tween.start()
}
const setAnimationCamera = (start, end) => {
  const tween = new TWEEN.Tween(start).to(end, 3000).easing(TWEEN.Easing.Quadratic.Out)
  tween.onUpdate((that) => {
      //  camera.postition  和 controls.target 一起使用
      camera.position.set(that.cx, that.cy, that.cz)
      controls.target.set(that.ox, that.oy, that.oz)
  })
  tween.start()
}

// 设置点击打开车门的动画效果
window.addEventListener('click', onPointClick);
function onPointClick(event) {
  let pointer = {}
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = - (event.clientY / window.innerHeight) * 2 + 1;
  var vector = new THREE.Vector2(pointer.x, pointer.y)
  var raycaster = new THREE.Raycaster()
  raycaster.setFromCamera(vector, camera)
  let intersects = raycaster.intersectObjects(scene.children);
  intersects.forEach((item) => {
    if (item.object.name === 'Object_64' || item.object.name === 'Object_77') {
      if (!carStatus || carStatus === 'close') {
        carLeftOpen()
        carRightOpen()
      } else {
        carLeftClose()
        carRightClose()
      }
    }
  })
}

// 创建聚光灯函数
const createSpotlight = (color) => {
  const newObj = new THREE.SpotLight(color, 2);
  newObj.castShadow = true;
  newObj.angle = Math.PI / 6;;
  newObj.penumbra = 0.2;
  newObj.decay = 2;
  newObj.distance = 50;
  return newObj;
}

// 设置图片背景
const spotLight1 = createSpotlight('#ffffff');
const texture = new THREE.TextureLoader().load('src/assets/imgs/奥特曼.jpg')
spotLight1.position.set(0, 3, 0);
spotLight1.target.position.set(-10, 3, 10)
spotLight1.map = texture
const lightHelper = new THREE.SpotLightHelper(spotLight1);
scene.add(spotLight1);

// 设置渲染函数
const render = (time) =>{ 
  controls.update()
  TWEEN.update(time)
  renderer.render(scene,camera)
  requestAnimationFrame(render)
}
render()
</script>