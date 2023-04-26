<template>
  <div>
    <q-btn-group>
      <q-btn
        :class="btnsToggleBool[0] ? 'pressed_btn' : ''"
        @click="toggleGizmo(0, 'positionGizmoEnabled')"
        glossy
        label="Смещение"
      />

      <q-btn
        :class="btnsToggleBool[1] ? 'pressed_btn' : ''"
        @click="toggleGizmo(1, 'rotationGizmoEnabled')"
        glossy
        label="Вращение"
      />

      <q-btn
        :class="btnsToggleBool[2] ? 'pressed_btn' : ''"
        @click="toggleGizmo(2, 'scaleGizmoEnabled')"
        glossy
        label="Масштабирование"
      />

      <q-btn
        :class="btnsToggleBool[3] ? 'pressed_btn' : ''"
        @click="toggleGizmo(3, 'boundingBoxGizmoEnabled')"
        glossy
        label="Мульти"
      />

      <q-btn
        :class="btnsToggleBool[4] ? 'pressed_btn' : ''"
        @click="toggleGizmo(4)"
        glossy
        label="Курсор"
      />
    </q-btn-group>

    <canvas ref="renderCanvas" id="renderCanvas" touch-action="none"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch } from "vue";

import * as BABYLON from "@babylonjs/core/Legacy/legacy";
const { pages } = defineProps({
  pages: Array,
});

let btnsToggleBool = ref([true, false, false, false, false]);

const renderCanvas = ref(null);

const gizmoManager = ref(null);

const selectedSphereRef = ref(null);

const currentGizmoAction = ref("positionGizmoEnabled");

const toggleGizmo = (ind, giz) => {
  btnsToggleBool.value = [false, false, false, false, false];
  btnsToggleBool.value[ind] = true;
  if (ind == 4) {
    toggleGizmoManager([false, false, false, false, false]);
    return;
  }
  if (gizmoManager.value) {
    gizmoManager.value[currentGizmoAction.value] = false;
    gizmoManager.value[giz] = !gizmoManager.value[giz];
  }
  currentGizmoAction.value = giz;
};

const toggleGizmoManager = function (arrValues) {
  gizmoManager.value.positionGizmoEnabled = arrValues[0];
  gizmoManager.value.rotationGizmoEnabled = arrValues[1];
  gizmoManager.value.scaleGizmoEnabled = arrValues[2];
  gizmoManager.value.boundingBoxGizmoEnabled = arrValues[3];
};
onMounted(() => {
  const canvas = renderCanvas.value;

  const engine = new BABYLON.Engine(canvas);

  let scene = new BABYLON.Scene(engine);

  let camera = new BABYLON.FreeCamera(
    "camera1",
    new BABYLON.Vector3(0, 5, -5),
    scene
  );
  camera.setTarget(BABYLON.Vector3.Zero());
  camera.attachControl(canvas, true);

  let light = new BABYLON.DirectionalLight(
    "light",
    new BABYLON.Vector3(0, -0.5, 1.0),
    scene
  );
  light.position = new BABYLON.Vector3(0, 5, -2);

  let spheres = [];

  for (let i = 0; i < 5; i++) {
    let sphere = BABYLON.Mesh.CreateSphere(`sphere${i}`, 16, 2, scene);
    sphere.material = new BABYLON.StandardMaterial("sphere material", scene);
    sphere.position.z = i;
    spheres.push(sphere);
    sphere.actionManager = new BABYLON.ActionManager(scene);
    sphere.material.diffuseColor = BABYLON.Color3.Blue();
    sphere.uniqField = "uniq";
  }

  let selected = null;
  scene.onPointerObservable.add(function (evt) {
    if (selected) {
      selected.material.diffuseColor = BABYLON.Color3.Blue();
      selected = null;
    }

    if (
      evt.pickInfo.hit &&
      evt.pickInfo.pickedMesh &&
      evt.event.button === 0 &&
      evt.pickInfo.pickedMesh?.uniqField
    ) {
      selected = evt.pickInfo.pickedMesh;
      evt.pickInfo.pickedMesh.material.diffuseColor = BABYLON.Color3.Green();
      toggleGizmoManager(btnsToggleBool.value);
    }

    if (!evt.pickInfo.pickedMesh?.uniqField) {
      toggleGizmoManager([false, false, false, false]);
    }
  }, BABYLON.PointerEventTypes.POINTERUP);

  gizmoManager.value = new BABYLON.GizmoManager(scene);
  gizmoManager.value.positionGizmoEnabled = true;

  gizmoManager.value.attachableMeshes = spheres;

  let ground = BABYLON.MeshBuilder.CreateGround(
    "ground1",
    {
      width: 12,
      height: 12,
      subdivisions: 2,
    },
    scene
  );
  ground.position.y = -1;
  scene.createDefaultXRExperienceAsync({ floorMeshes: [ground] });

  engine.runRenderLoop(() => {
    scene.render();
  });
});
</script>

<style scoped lang="scss">
html,
body {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  overflow: hidden;
}
.pressed_btn {
  background-color: rgb(255, 209, 6);
  box-shadow: inset 0px 0px 5px rgba(0, 0, 0, 0.2);
  transform: translateY(2px);
}
#renderCanvas {
  width: 100%;
  height: 100%;
  display: block;
  font-size: 0;
}
</style>
