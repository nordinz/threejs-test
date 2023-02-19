<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as dat from "dat.gui";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import vectorOne from "../assets/img/vectorOne.jpg";
import vectorTwo from "../assets/img/vectorTwo.jpg";

const bjUrl = new URL("../assets/img/bj.glb", import.meta.url);

const renderer = new THREE.WebGL1Renderer();

renderer.shadowMap.enabled = true;

renderer.setSize(window.innerWidth, window.innerHeight);

document.body.appendChild(renderer.domElement);

// skapa en "scen"
const scene = new THREE.Scene();

// skapa en kamera av typen "perspektiv"
const camera = new THREE.PerspectiveCamera(
  75, // field of view
  window.innerWidth / window.innerHeight, // aspect
  0.1, // near clipping
  1000 // far clipping
);

const orbit = new OrbitControls(camera, renderer.domElement); //skapa "orbitControl" så vi kan styra kameran med musen

// const axesHelper = new THREE.AxesHelper(3); // skapa till en guide för att se kordinater, arg = längden på axlarna
// scene.add(axesHelper); // lägg till den i scenen

camera.position.set(-10, 30, 30); //arg1 = x axis, arg2 = y axis, arg3 = z axis
orbit.update(); //uppdatera urbit varje gång vi rör på kameran

//skapa text

//skapa en 3d box
// const boxGeometry = new THREE.BoxGeometry(); //skapa box-skelettet
// const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 }); // skapa ytan/fäärgen
// const box = new THREE.Mesh(boxGeometry, boxMaterial); // sätt ihop skelett och yta/färg
// scene.add(box); // lägg till allt till scenen

// //skapa en "plane" en platta/husgrund/"markplan"
// const planeGeometry = new THREE.PlaneGeometry(30, 30); // skelett
// const planeMaterial = new THREE.MeshStandardMaterial({
//   color: 0xffffff, //färg
//   side: THREE.DoubleSide, //ge "plane" en undersida
// }); //färg/yta
// const plane = new THREE.Mesh(planeGeometry, planeMaterial); // sätt ihop båda
// scene.add(plane); // skicka ut i scenen(skärmen)
// plane.rotation.x = -0.5 * Math.PI; // rotera plane så den lägger sig på griden
// plane.receiveShadow = true;

// const gridHelper = new THREE.GridHelper(30); //lägg till en gridhelper och lägg till värde som arg för att utöka storlek på grid, arg2 = mindre rutor
// scene.add(gridHelper);

//skapa en sphere
const sphereGeometry = new THREE.SphereGeometry(4, 50, 50);
const sphereMaterial = new THREE.MeshStandardMaterial({
  color: 0x0000ff,
  wireframe: false, // visa objektets skelett
});
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
scene.add(sphere);

sphere.position.set(-10, 10, 0);
sphere.castShadow = true;

const ambientLight = new THREE.AmbientLight(0x333333);
scene.add(ambientLight);

// const directinalLight = new THREE.DirectionalLight(0xffffff, 0.8);
// scene.add(directinalLight);
// directinalLight.position.set(-30, 50, 0);
// directinalLight.castShadow = true;
// directinalLight.shadow.camera.bottom = -12;

// const dLightHelper = new THREE.DirectionalLightHelper(directinalLight);
// scene.add(dLightHelper);

// const dLightShadowHelper = new THREE.CameraHelper(
//   directinalLight.shadow.camera
// );
// scene.add(dLightShadowHelper);

const spotLight = new THREE.SpotLight(0xffffff);
scene.add(spotLight);
spotLight.position.set(-100, 100, 0);
spotLight.castShadow = true;
spotLight.angle = 0.14;

// const sLightHelper = new THREE.SpotLightHelper(spotLight);
// scene.add(sLightHelper);

scene.fog = new THREE.FogExp2(0xffffff, 0.01);

// lägg solid backgrund
// renderer.setClearColor(0xffea00);

const textureLoader = new THREE.TextureLoader();
// scene.background = textureLoader.load(vectorOne); // en solid 2d bild som bg
const cubeTextureLoader = new THREE.CubeTextureLoader();
scene.background = cubeTextureLoader.load([
  vectorOne,
  vectorTwo,
  vectorOne,
  vectorOne,
  vectorOne,
  vectorOne,
]);

const box2Geometry = new THREE.BoxGeometry(4, 4, 4);
const box2Material = new THREE.MeshBasicMaterial({
  // color: 0x00ff00,
  // map: textureLoader.load(vectorOne),
});
const box2MultiMaterial = [
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorOne) }),
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorOne) }),
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorOne) }),
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorTwo) }),
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorTwo) }),
  new THREE.MeshBasicMaterial({ map: textureLoader.load(vectorTwo) }),
];
const box2 = new THREE.Mesh(box2Geometry, box2MultiMaterial);
scene.add(box2);
box2.position.set(0, 15, 10);
// box2.material.map = textureLoader.load(vectorTwo);
box2.castShadow = true;

const assetLoader = new GLTFLoader();

assetLoader.load(
  bjUrl.href,
  function (gltf) {
    const model = gltf.scene;
    scene.add(model);
    model.position.set(-9, 4, 10);
  },
  undefined,
  function (error) {
    console.log(error);
  }
);

const gui = new dat.GUI();
const options = {
  sphereColor: "#ffea00",
  wireframe: false,
  speed: 0.01,
  angle: 0.15,
  penumbra: 0,
  intensity: 1,
};
gui.addColor(options, "sphereColor").onChange(function (e) {
  sphere.material.color.set(e);
});
gui.add(options, "wireframe").onChange(function (e) {
  sphere.material.wireframe = e;
});

gui.add(options, "speed", 0, 0.1);

gui.add(options, "angle", 0, 1);
gui.add(options, "penumbra", 0, 1);
gui.add(options, "intensity", 0, 1);

let step = 0;

const mousePosition = new THREE.Vector2();

window.addEventListener("mousemove", function (e) {
  mousePosition.x = (e.clientX / window.innerWidth) * 2 - 1;
  mousePosition.y = -(e.clientY / window.innerHeight) * 2 + 1;
});

const rayCaster = new THREE.Raycaster();

const sphereId = sphere.id;
box2.name = "theBox";

// animera boxen(rotera runt dess axis)
function animate(time) {
  //sätt time som arg för att kontrollera hastighet nedan
  // box.rotation.x = time / 1000;
  // box.rotation.y = time / 1000;

  step += options.speed;
  sphere.position.y = 10 * Math.abs(Math.sin(step));

  spotLight.angle = options.angle;
  spotLight.penumbra = options.penumbra;
  spotLight.intensity = options.intensity;
  // sLightHelper.update();

  rayCaster.setFromCamera(mousePosition, camera);
  const intersects = rayCaster.intersectObjects(scene.children);
  // console.log(intersects);

  for (let i = 0; i < intersects.length; i++) {
    if (intersects[i].object.id === sphereId)
      intersects[i].object.material.color.set(0xff0000);

    if (intersects[i].object.name === "theBox") {
      intersects[i].object.rotation.x = time / 1000;
      intersects[i].object.rotation.y = time / 1000;
    }
  }

  renderer.render(scene, camera);
  //console.log(window.innerHeight);
}
renderer.setAnimationLoop(animate);

// renderer.render(scene, camera); // länka scenen med kameran, in med scene och kamera som argument

window.addEventListener("resize", function () {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

<template>
  <body></body>
</template>
<style scoped>
body {
  margin: 0;
}
</style>
