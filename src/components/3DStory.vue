<template>
  <div class="modal" v-show="data.modalVisible">
    <div class="playGame">
      <div class="btn" @click="toggleContent(0)">开始操作</div>
      <audio ref="audioPlayer" loop autoplay src="../../public/sounds/LOVE.mp3"></audio>
    </div>
  </div>
  <div class="textDiv" v-show="data.contentVisible">
    <div class="text">{{ data.contentList[data.index].content }}</div>
    <div class="footer">
      <div
        v-for="(item, i) in data.contentList[data.index].btns"
        class="btn"
        @click="toggleContent(item.index)"
      >
        {{ item.name }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref ,reactive, onMounted } from 'vue'
// 导入threejs
import * as THREE from "three";
// 导入轨道控制器
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
// 导入TWEEN
import * as TWEEN from "tween";

// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  45, // 视角
  window.innerWidth / window.innerHeight, // 宽高比
  0.1, // 近平面
  1000 // 远平面
);
// 设置相机位置
camera.position.z = 1;
// camera.position.y = 2;
// camera.position.x = 2;
camera.lookAt(0, 0, 0);

// 创建渲染器
const renderer = new THREE.WebGLRenderer({
  antialias: true, // 开启抗锯齿
});
renderer.shadowMap.enabled = true; // 启用阴影映射
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 添加轨道控制器
const controls = new OrbitControls(camera, renderer.domElement);
// 设置带阻尼的惯性
controls.enableDamping = true;
// 设置阻尼系数
controls.dampingFactor = 0.05;
controls.maxDistance = 50;
// 设置旋转速度
// controls.autoRotate = true;

// 渲染函数
const animate = () => {
  controls.update();
  TWEEN.update();
  requestAnimationFrame(animate);
  // 渲染
  renderer.render(scene, camera);
}
animate();


let data = reactive({
  contentList: [
    {
      content:
        "阿伟坐在电脑前，一边打游戏，一边听着妈妈的唠叨。他的脸上满是不耐烦，心中充满了对妈妈的反感。此时，他的朋友小刚走进来，邀请他一起去网吧游玩。阿伟欣然答应，两人一起出门。",
      img: "./textures/story/1.jpg",
      sound: "./sounds/1.mp3",
      startAngle: { x: 0, y: 0 },
      endAngle: { x: -Math.PI / 8, y: Math.PI / 2 },
      duration: 15000,
      btns: [
        {
          name: "不能听妈妈的唠叨，我决定必须和朋友出去玩~",
          index: 1,
        },
        {
          name: "阿伟回头想了想，现在是学习的关键时刻，不能老是沉迷于游戏。悬崖勒马回头是岸！",
          index: 2,
        },
      ],
    },
    {
      content:
        "阿伟和小刚在网吧里玩得不亦乐乎，他们在游戏中大显身手，引来了众人的羡慕目光。下机后，他们准备离开，却被一位名叫杰哥的人叫住。",
      img: "./textures/story/2.jpg",
      sound: "./sounds/2.mp3",
      startAngle: { x: Math.PI / 16, y: Math.PI - Math.PI / 16 },
      endAngle: { x: Math.PI / 16, y: Math.PI + Math.PI / 16 },
      duration: 20000,
      btns: [
        {
          name: "是要发生什么事吗...",
          index: 3,
        },
      ],
    },
    {
      content:
        "阿伟和妈妈重新回到了宁静的生活，他们学会了如何面对生活中的困境和挑战，也更加珍惜彼此之间的感情。",
      img: "./textures/story/3.jpg",
      sound: "./sounds/3.mp3",
      startAngle: { x: 0, y: -Math.PI / 4 },
      endAngle: { x: 0, y: -Math.PI / 2 },
      duration: 25000,
      btns: [],
    },
    {
      content:
        "杰哥热情地邀请阿伟和小刚到他家玩，他们在欢笑声中喝得烂醉如泥。杰哥看着阿伟，眼神中闪烁着诡异的光芒。",
      img: "./textures/story/4.jpg",
      sound: "./sounds/4.mp3",
      startAngle: { x: Math.PI / 16, y: -Math.PI / 2 - Math.PI / 8 },
      endAngle: { x: Math.PI / 16, y: -Math.PI / 2 },
      duration: 20000,
      btns: [
        {
          name: "阿伟被半推半就的被杰哥拉扯着...",
          index: 4,
        },
      ],
    },
    {
      content:
        "杰哥把阿伟带到他的房间，让他坐在桌前。阿伟的视线落在桌上，他看到了一些他从未见过的物品，他的心跳开始加速。",
      img: "./textures/story/5.jpg",
      sound: "./sounds/5.mp3",
      startAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 4 },
      endAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 8 },
      duration: 25000,
      btns: [
        {
          name: "这些到底是什么...",
          index: 5,
        },
      ],
    },
    {
      content:
        "杰哥趁阿伟脸红的时候，想看他法语正不正常。阿伟感到有些不安，但他无法反抗。杰哥一拳把他打到床上，他无力反抗，只能任由杰哥为所欲为。",
      img: "./textures/story/6.jpg",
      sound: "./sounds/6.mp3",
      startAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 4 },
      endAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 8 },
      duration: 25000,
      btns: [
        {
          name: "事后...",
          index: 6,
        },
      ],
    },
    {
      content:
        "杰哥笑着对阿伟说：“我是阳光dua郎大男孩，这是我们的秘密你别给我说出去。”阿伟无奈地点头，心中充满了恐惧和无助。第二天阿伟收到了杰哥发来的消息，说依然想他再来他家开party。阿伟心中充满了恐惧，他知道，他已经陷入了一个无法逃脱的深渊。",
      img: "./textures/story/7.jpg",
      sound: "./sounds/7.mp3",
      startAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 4 },
      endAngle: { x: Math.PI / 16, y: Math.PI / 2 - Math.PI / 8 },
      duration: 25000,
      btns: [
        {
          name: "阿伟：我不能就这么完了~",
          index: 7,
        },
      ],
    },
    {
      content:
        "阿伟决定向警察求助，他要揭露杰哥的罪行，让他得到应有的惩罚。他知道，这将是一场艰难的战斗，但他没有退路，他必须站出来，为自己和其他可能成为杰哥目标的人争取公正。",
      img: "./textures/story/8.jpg",
      sound: "./sounds/8.mp3",
      startAngle: { x: 0, y: -Math.PI / 2 - Math.PI / 4 },
      endAngle: { x: -Math.PI / 8, y: -Math.PI / 2 - Math.PI / 8 },
      duration: 25000,
      btns: [],
    },
  ],
  contentVisible: false,
  modalVisible: true,
  index: 0,
});

let textureLoader = new THREE.TextureLoader();
let textures = data.contentList.map((item, i) => {
  let texture = textureLoader.load(data.contentList[i].img); // 循环加载每一张图片
  texture.mapping = THREE.EquirectangularReflectionMapping; // 通过使用全景纹理图像来模拟环境反射
  texture.colorSpace = THREE.SRGBColorSpace; // 表示和描述颜色的数学模型或系统
  return texture;
});
let SphereGeometry = new THREE.SphereGeometry(100, 32, 32);
SphereGeometry.scale(1, 1, -1);
let material = new THREE.MeshBasicMaterial({ map: textures[data.index] });
let sphere = new THREE.Mesh(SphereGeometry, material);
sphere.rotation.order = "XYZ";
scene.add(sphere);

let audio = new Audio();
let tween;
let audioPlayer = ref(null);

function toggleContent(dataIndex) {
  audioPlayer.value.play();
  setTimeout(() => {
    data.contentList[data.index].sound &&
      (audio.src = data.contentList[data.index].sound);
    audio.play();
  }, 500);
  data.contentVisible = true;
  data.modalVisible = false;
  data.index = dataIndex;
  camera.position.set(0, 0, 1);
  sphere.rotation.y = data.contentList[dataIndex].startAngle.y;
  sphere.rotation.x = data.contentList[dataIndex].startAngle.x;
  material.map = textures[data.index];
  material.needsUpdate = true;

  tween && tween.stop();
  tween = new TWEEN.Tween(sphere.rotation);
  tween.to(
    {
      y: data.contentList[data.index].endAngle.y,
      x: data.contentList[data.index].endAngle.x,
    },
    data.contentList[data.index].duration
  );
  // 设置缓动函数
  tween.easing(TWEEN.Easing.Quadratic.InOut);

  // 启动补间动画
  tween.start();
}

</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
body {
  width: 100vw;
  height: 100vh;
}
canvas {
  display: block;
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}

.textDiv {
  width: 80%;
  max-width: 500px;
  height: auto;
  padding: 20px 50px;
  border-radius: 10px;
  border: 1px solid #9999cc;
  box-shadow: 0 0 5px #ddddff;
  z-index: 100;
  position: fixed;
  left: 50%;
  bottom: 30px;
  transform: translate(-50%, 0);
  color: #ffffff;
  background-color: rgba(0, 0, 0, 0.5);
  text-align: left;
  line-height: 25px;
}
.playGame {
  width: 800px;
  height: 450px;
  background-image: url(../assets/imgs/boys.jpg);
  background-size: 100% 100%;
  border-radius: 50px;
  position: absolute;
  left: calc(50% - 400px);
  top: calc(50% - 225px);
  z-index: 100;
}
.modal {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
  z-index: 100;
  background-color: rgba(0, 0, 0, 0.9);
}
.playGame .btn {
  width: 200px;
  height: 50px;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
  position: absolute;
  left: calc(50% - 100px);
  bottom: 50px;
  cursor: pointer;
}
.playGame .btn:hover {
  background: red;
}
.textDiv .footer {
  display: flex;
  justify-content: end;
  padding: 15px 0;
  flex-direction: column;
  align-items: start;
}
.textDiv .btn {
  width: auto;
  height: auto;
  background-color: rgba(50, 50, 100, 0.5);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
  font-size: 12px;
  padding: 5px 10px;
  margin-bottom: 10px;
  line-height: 24px;
  cursor: pointer;
}
.textDiv .btn:hover {
  background: red
}
</style>