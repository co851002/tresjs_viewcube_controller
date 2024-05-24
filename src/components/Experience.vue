<template>
  <div id="experience">
    <TresCanvas v-bind="glSettings">
      <!-- <TresOrthographicCamera ref="cameraRef" :position="[10, 20, 20]" />  -->
      <TresPerspectiveCamera ref="cameraRef" :position="[10, 20, 20]" />
      <OrbitControls make-default />
      <TresAmbientLight :intensity="0.1" />
      <TresDirectionalLight color="red" :position="[0, 0, 1]" />
      <Scene :initialViewCubeController="viewCubeController" />
    </TresCanvas>
    <div id="viewcube-container">
      <div class="cube">
        <div
          v-for="orientation in cubeOrientations"
          :key="orientation"
          :class="`cube__face cube__face--${orientation.toLowerCase()}`"
          @click="setOrientation(orientation)"
        >
          {{ orientation }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { shallowRef, ref, onMounted, defineComponent, nextTick } from "vue";
import { TresCanvas } from "@tresjs/core";
import { OrbitControls } from "@tresjs/cientos";
import * as THREE from "three";
import { STLLoader } from "three/examples/jsm/loaders/STLLoader";
import TWEEN from "@tweenjs/tween.js";
import ViewCubeController from "../viewcube.ts";

const glSettings = {
  clearColor: "#ddd",
  shadows: true,
  alpha: false,
  shadowMapType: THREE.BasicShadowMap,
  outputColorSpace: THREE.SRGBColorSpace,
  toneMapping: THREE.NoToneMapping,
  windowSize: true,
};

const cameraRef = shallowRef(null);
const viewCubeController = shallowRef(null);
const cubeOrientations = ["Top", "Bottom", "Front", "Back", "Left", "Right"];

function setOrientation(orientation) {
  if (viewCubeController.value) {
    viewCubeController.value.tweenCamera(
      ViewCubeController.ORIENTATIONS[
        ViewCubeController.CubeOrientation[orientation]
      ]
    );
  }
}

function getCameraCSSMatrix(matrix) {
  const { elements } = matrix;

  return `matrix3d(
      ${epsilon(elements[0])},
      ${epsilon(-elements[1])},
      ${epsilon(elements[2])},
      ${epsilon(elements[3])},
      ${epsilon(elements[4])},
      ${epsilon(-elements[5])},
      ${epsilon(elements[6])},
      ${epsilon(elements[7])},
      ${epsilon(elements[8])},
      ${epsilon(-elements[9])},
      ${epsilon(elements[10])},
      ${epsilon(elements[11])},
      ${epsilon(elements[12])},
      ${epsilon(-elements[13])},
      ${epsilon(elements[14])},
      ${epsilon(elements[15])})`;
}

function epsilon(value) {
  return Math.abs(value) < 1e-10 ? 0 : value;
}

onMounted(() => {
  nextTick(() => {
    if (cameraRef.value) {
      viewCubeController.value = new ViewCubeController(cameraRef.value);
      animate();
    }
  });
});

function animate() {
  requestAnimationFrame(animate);
  TWEEN.update();
  updateCubeTransformation();
}

function updateCubeTransformation() {
  const cube = document.querySelector(".cube");
  const camera = cameraRef.value;

  if (cube && camera) {
    const mat = new THREE.Matrix4();
    mat.extractRotation(camera.matrixWorldInverse);
    cube.style.transform = `translateZ(-300px) ${getCameraCSSMatrix(mat)}`;
  }
}

function useModel() {
  const stl = shallowRef(null);
  const loader = new STLLoader();

  loader.load("/teapot.stl", (geometry) => {
    const material = new THREE.MeshStandardMaterial({ color: "hotpink" });
    const mesh = new THREE.Mesh(geometry, material);
    mesh.position.set(0, -2.5, 0);
    mesh.rotation.x = -Math.PI / 2;
    stl.value = mesh;
  });

  return stl;
}

const Scene = defineComponent({
  props: ["initialViewCubeController"],
  setup(props) {
    const stl = useModel();
    const localViewCubeController = shallowRef(props.initialViewCubeController);
    console.log(stl);

    onMounted(() => {
      if (!localViewCubeController.value && cameraRef.value) {
        localViewCubeController.value = new ViewCubeController(cameraRef.value);
      }
    });

    return { stl, localViewCubeController };
  },
  template: `
      <TresObject3D>
        <primitive v-if="stl" :object="stl" :scale="[1, 1, 1]" />
      </TresObject3D>
    `,
});
</script>

<style>
#experience {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

#viewcube-container {
  bottom: 40px;
  height: 120px;
  margin: 10px;
  -webkit-perspective: 600px;
  perspective: 600px;
  position: absolute;
  right: 60px;
  width: 120px;
  z-index: 2;
}

.cube {
  width: 80px;
  height: 80px;
  transform-style: preserve-3d;
}

.cube__face {
  position: absolute;
  width: 120px;
  height: 120px;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid #333;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 22px;
  color: #333;
  font-size: 25px;
  font-weight: 700;
}

.cube__face--top {
  transform: rotateY(0deg) rotateX(90deg) translateZ(-60px);
}
.cube__face--bottom {
  transform: rotateX(270deg) translateZ(-60px);
}
.cube__face--left {
  transform: rotateY(-90deg) rotateX(180deg) rotateZ(0deg) translateZ(-60px);
}
.cube__face--right {
  transform: rotateY(90deg) rotateX(180deg) rotateZ(0deg) translateZ(-60px);
}
.cube__face--front {
  transform: rotateX(180deg) translateZ(-60px);
}
.cube__face--back {
  transform: rotateZ(180deg) translateZ(-60px);
}
.cube__face:hover {
  background: #adadad;
  color: #fff;
}
</style>
