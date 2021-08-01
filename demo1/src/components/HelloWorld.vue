<template>
  <div>
    <div id="container" ref="container"></div>
  </div>
</template>

<script>
import * as THREE from 'three';
import TWEEN from 'tween'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return{
      // bigImg: require("@/assets/logo.png"), // 图片路径
      bigImg: require("@/assets/vr.jpg"), // 图片路径
      container: null, // 页面容器
      camera: null, // 相机
      renderer: null, // 渲染器
      scene: null, // 场景
      material: null,  // 添加材质
      texture: null,// 创建纹理贴图
      skyBox: null, // 网格
      controls: null, // 轨道控制
      clock: null, // 轨道更新时间
      // 鼠标属性
      bMouseDown: false,
      x: -1,
      y: -1,
      isClickCamera: false, // 是否点运动相机
      raycaster: null,
      mouse: null,
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.init();
      this.animate();
    })
  },
  methods: {
    // 初始化轨道控制
    initControls() {
        this.controls = new OrbitControls(this.camera, this.renderer.domElement);
        this.controls.target = new THREE.Vector3(0, 0, 0);
        this.controls.minDistance = 18; // 相机最近
        this.controls.maxDistance = 500; // 相机最远
        this.controls.autoRotate = false; // 图片自动旋转
        this.controls.enableDamping = true; // 使动画循环使用时阻尼或自转 意思是否有惯性
        this.controls.enablePan = false; // 是否开启右键拖拽
        this.controls.autoRotateSpeed = 0.5; // 阻尼系数
    },
    init() {
        // 页面容器
        this.container = document.getElementById('container');

        // 创建渲染器
        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setPixelRatio(window.devicePixelRatio);

        // 设置画布的宽高
        this.renderer.setSize(window.innerWidth, window.innerHeight);

        // 判断容器中子元素的长度
        let childs = this.container.childNodes;
        if (this.container.childNodes.length > 0) {
            this.container.removeChild(childs[0]);
            this.container.appendChild(this.renderer.domElement);
        } else {
            this.container.appendChild(this.renderer.domElement);
        }
        // container.appendChild(renderer.domElement);

        // 创建场景
        this.scene = new THREE.Scene();

        // 创建相机
        this.camera = new THREE.PerspectiveCamera(80, window.innerWidth / window.innerHeight, 1, 10000);
        this.camera.position.set(5, 0, 0);
        this.camera.lookAt(new THREE.Vector3(0, 0, 0)); //让相机指向原点 

        // 创建轨道控制器
        this.initControls();

        // 添加材质
        this.material = new THREE.MeshBasicMaterial();
        // 创建纹理贴图
        this.texture = new THREE.TextureLoader().load(this.bigImg);
        this.material.map = this.texture;

        // 创建网格对象
        this.skyBox = new THREE.Mesh(new THREE.SphereBufferGeometry(100, 100, 100), this.material);
        this.skyBox.geometry.scale(1, 1, -1);
        // 显示坐标光线
        var axisHelper = new THREE.AxisHelper(600); // 显示光线（红色代表X轴，绿色代表Y轴，蓝色代表Z轴）
        // 添加到场景中去
        this.scene.add(axisHelper);
        this.scene.add(this.skyBox);
    
        // 鼠标事件监听
        this.renderer.domElement.addEventListener('pointerdown', this.onMouseDown, false);
        this.renderer.domElement.addEventListener('pointerup', this.onMouseUp, false);
        this.renderer.domElement.addEventListener('pointermove', this.onMouseMove, false);

        // 监听布局变化
        window.addEventListener('resize', this.onWindowResize, false);

        
    },
      // 更新相机动画
    tweenCamera(position, target) {
        new TWEEN.Tween(this.camera.position).to({
            x: position.x,
            y: position.y,
            z: position.z
        }, 600).easing(TWEEN.Easing.Sinusoidal.InOut).start();

        new TWEEN.Tween(this.controls.target).to({
            x: target.x,
            y: target.y,
            z: target.z
        }, 600).easing(TWEEN.Easing.Sinusoidal.InOut).start();
    },
    // 鼠标按下
    onMouseDown(event) {
        event.preventDefault(); // 取消默认事件
        console.log("---onMouseDown---");
        this.isClickCamera = true;
    },
    // 鼠标放开
    onMouseUp(event) {
        event.preventDefault(); // 取消默认事件
        console.log("---onMouseUp---");
        if (this.isClickCamera) {
            console.log("---移动相机---", event);
            // 红色代表X轴，绿色代表Y轴，蓝色代表Z轴
            this.mouse = new THREE.Vector3(); // 三维坐标对象
            // 屏幕坐标到标准化设备坐标(Normalized Device Coordinates, NDC)转换
            this.mouse.set((event.clientX / window.innerWidth ) * 2 - 1, - (event.clientY / window.innerHeight ) * 2 + 1, 0.5 );
            this.mouse.unproject(this.camera);

            this.raycaster = new THREE.Raycaster(this.camera.position, this.mouse.sub(this.camera.position).normalize()); // 投手
            var intersects = this.raycaster.intersectObjects(this.scene.children);
            
            if (intersects.length > 0) {
                var selected = intersects[0]; // 取第一个物体
                console.log("x坐标:"+ selected.point.x);
                console.log("y坐标:"+ selected.point.y);
                console.log("z坐标:"+ selected.point.z);

                // var direction = new THREE.Vector3();
                // this.camera.getWorldDirection( direction );
                // this.camera.position.add( direction );
                // this.camera.position.add(direction.multiplyScalar(selected.point.x, selected.point.y, selected.point.z) );
                // this.camera.lookAt(new THREE.Vector3(selected.point.x, selected.point.y, selected.point.z)); //让相机指向原点 
                this.camera.position.set(selected.point.x, selected.point.y, selected.point.z);

                // this.tweenCamera(selected.point, selected.point);
            }
        }
    },
    // 鼠标移动
    onMouseMove(event) {
        event.preventDefault(); // 取消默认事件
        console.log("---onMouseMove---");
        this.isClickCamera = false;
    },
    onWindowResize() {
        // 窗口缩放的时候,保证场景也跟着一起缩放
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    animate() {
        requestAnimationFrame(this.animate);
        this.controls.update(); // 更新轨道控制
        TWEEN.update();
        this.renderer.render(this.scene, this.camera);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
