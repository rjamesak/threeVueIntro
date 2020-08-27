<template>
  <div class="three-d-world" ref="three"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
// import { BasisTextureLoader } from "three/examples/jsm/loaders/BasisTextureLoader";

export default {
  name: "threeDWorld",
  methods: {
    init() {
      console.log("init method");
      this.cameraSetup();
      this.setRenderer();
      this.getBaccMaterial();
      // this.getSkullMaterial();
      this.addGeometry();
      this.addModel();
      this.addSkullModel();
      // this.addGround();
      this.setControls();
      this.initRaycaster();
      this.eventListener();
      this.animate();
    },
    //create the camera and the scene
    cameraSetup() {
      //camera
      this.camera = new THREE.PerspectiveCamera(
        50,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      //create scene
      this.scene = new THREE.Scene();
      this.camera.position.set(5, 6, 200);
    },
    setRenderer() {
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      // this.renderer.outputEncoding = THREE.sRGBEncoding;
      this.$refs.three.appendChild(this.renderer.domElement);
    },
    setControls() {
      console.log("in controls");
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      // this.controls.enableZoom = true;
      // this.controls.enablePan = true;
      // this.controls.enableDamping = true;
      // this.controls.rotateSpeed = -0.25;
    },
    getBaccMaterial() {
      this.loader = new THREE.TextureLoader();
      this.baccMaterial = null;
      this.loader.load(
        "http://localhost:8000/BaccTex/Bacticinerator2FBX_Main_Normal.png",
        (texture) => {
          console.log("in material loader");
          this.baccMaterial = new THREE.MeshBasicMaterial({
            map: texture,
          });
        },
        undefined,
        (err) => {
          console.log("err in getMaterial: ", err);
        }
      );
    },
    getSkullMaterial() {
      this.skullMaterial = null;
      this.loader.load(
        "http://localhost:8000/skullModel/textures/defaultMat_baseColor.jpeg",
        (texture) => {
          this.skullMaterial = new THREE.MeshBasicMaterial({
            map: texture,
          });
          console.log("skull texture: ", texture);
          texture.encoding = THREE.sRGBEncoding;
          texture.flipY = false;
        },
        undefined,
        (err) => {
          console.log("err get skullMaterial: ", err);
        }
      );
    },
    addGeometry() {
      this.geometry = new THREE.BoxBufferGeometry(50, 50, 50);
      // this.geom2 = new THREE.BoxBufferGeometry(50, 50, 50);
      //texture loading
      this.loader = new THREE.TextureLoader();
      console.log("loader created", this.loader);
      this.loader.load(
        //texture url
        // "https://i.imgur.com/O4qyCZ8.jpg",
        // "https://tr.rbxcdn.com/67f41b47949fd17d00450bc47cccfa45/420/420/Decal/Png",
        "http://localhost:8000/Wood-067_sm.png",
        //onload callback
        (texture) => {
          console.log("texture load success", texture);
          // texture.wrapS = THREE.RepeatWrapping;
          // texture.wrapT = THREE.RepeatWrapping;
          // texture.repeat.set(1, 4);
          this.material = new THREE.MeshBasicMaterial({
            map: texture,
            color: 0xffffff,
          });
          console.log("material in texture callback: ", this.material);
          this.cube = new THREE.Mesh(this.geometry, this.material);
          this.cube.position.y = 25;
          // this.cube2 = new THREE.Mesh(this.geom2, this.material);
          this.scene.add(this.cube);
          // this.scene.add(this.cube2);
        },
        //progress callback not supported
        undefined,

        //load error callback
        function (err) {
          console.error("error in texture loader", err);
        }
      );
      console.log("material outside callback", this.material);
      //MATERIAL
      // this.material = new THREE.MeshBasicMaterial({
      // wireframe: true,
      // color: 0xbc1b1b,
      // wireframeLinejoin: "miter",
      // map: this.texture,
      // envMap: "reflection",
      // });
    },
    addModel() {
      this.modelLoader = new FBXLoader();
      console.log("adding model");
      this.modelLoader.load(
        "http://localhost:8000/Bacticinerator2FBX.FBX",
        (object) => {
          console.log("adding the model object to the scene");
          console.log("model object: ", object);
          object.traverse((child) => {
            if (child.isMesh) {
              console.log("found mesh");
              child.material = this.baccMaterial;
            }
          });
          object.position.set(50, 0, 0);
          this.scene.add(object);
        }
      );
    },
    addSkullModel() {
      this.gltfLoader = new GLTFLoader();
      this.gltfLoader.load(
        "http://localhost:8000/skullModel/scene.gltf",
        (gltf) => {
          this.scene.add(gltf.scene);
          console.log("gltf: ", gltf);
          gltf.scene.position.set(50, 25, 0);
          gltf.scene.scale.set(10, 10, 10);
          //traverse with gltf.scene.traverse((callback Obj)=> {})
          gltf.scene.traverse((obj) => {
            if (obj.isMesh) {
              console.log("found mesh in gltf traverse: ", obj);
              // obj.material.color.setRGB(50, 100, 200);
              obj.material.emissive.setRGB(50, 100, 200);
            }
          });
        },
        undefined,
        (error) => {
          console.error(error);
        }
      );
    },
    addGround() {
      this.groundMesh = new THREE.Mesh(
        new THREE.PlaneBufferGeometry(2000, 2000),
        new THREE.MeshBasicMaterial({ color: 0x808080 })
      );
      this.groundMesh.rotation.x = -Math.PI / 2;
      this.scene.add(this.groundMesh);
    },
    addLights() {
      //lights
      this.directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      this.lightPosition = new THREE.Vector3(10, 10, 0);
      this.directionalLight.position = this.lightPosition;
      //shadows

      //add lights to scene
      this.scene.add(this.directionalLight);
    },
    initRaycaster() {
      console.log("raycaster init");
      this.raycaster = new THREE.Raycaster();
      this.mouse = new THREE.Vector2();
    },
    eventListener() {
      this.$refs.three.addEventListener("mousemove", this.onMouseMove, false);
      window.addEventListener("resize", this.handleResize, false);
    },
    onMouseMove(event) {
      // console.log("moving mouse");
      //calc mouse pos in normalized device coordinates
      //(-1 to 1) for both components
      this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    },
    handleResize() {
      console.log("resize fired");
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
    },
    animate() {
      requestAnimationFrame(this.animate);
      //update picking ray with camera and mouse pos
      this.raycaster.setFromCamera(this.mouse, this.camera);

      // console.log("this.mouse: ", this.mouse);
      // console.log("ray far: ", this.raycaster.far);
      // console.log("geom attributes: ", this.geometry.attributes);
      //calc objects intersecting the ray
      this.intersects = this.raycaster.intersectObjects(this.scene.children); //returns array
      //
      this.intersected = null;
      // for (var i = 0; i < this.intersects.length; i++) {
      //   this.intersected = this.intersects[i].object;
      //   this.intersects[i].object.material.color.set(0xff0000);
      //   console.log("hit");
      // }

      //Need a better way to change and revert the colors
      //make this better
      if (this.intersects.length > 0) {
        console.log("hit");
        // this.intersected = this.intersects[0].object;
        // this.intersects[0].object.material.color.set(0xff0000);
      } else {
        // this.material.color.set(0xffffff);
      }
      // console.log("this.intersected", this.intersected);

      // console.log("this.intersects.length: ", this.intersects.length);
      // console.log("this.intersected: ", this.intersected);
      // this.controls.update();

      this.renderer.render(this.scene, this.camera);
    },
  },
  mounted() {
    this.init();
  },
};
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
