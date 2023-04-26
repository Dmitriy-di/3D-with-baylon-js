<template>
  <div>
    <q-btn-group>
      <q-btn
        color="secondary"
        @click="toggleGizmo('positionGizmoEnabled')"
        glossy
        label="First"
      />
      <q-btn
        color="secondary"
        @click="toggleGizmo('rotationGizmoEnabled')"
        glossy
        label="Second"
      />
      <q-btn
        color="secondary"
        @click="toggleGizmo('scaleGizmoEnabled')"
        glossy
        label="Third"
      />
      <q-btn
        color="secondary"
        @click="toggleGizmo('boundingBoxGizmoEnabled')"
        glossy
        label="Fourth"
      />
    </q-btn-group>
    <canvas ref="renderCanvas" id="renderCanvas" touch-action="none"></canvas>
  </div>
</template>

<script setup>
import { FreeCamera } from "@babylonjs/core/Cameras/freeCamera";
import { Engine } from "@babylonjs/core/Engines/engine";
import { HemisphericLight } from "@babylonjs/core/Lights/hemisphericLight";
import { Vector3 } from "@babylonjs/core/Maths/math.vector";
import { CreateGround } from "@babylonjs/core/Meshes/Builders/groundBuilder";
import { CreateSphere } from "@babylonjs/core/Meshes/Builders/sphereBuilder";
import { Scene } from "@babylonjs/core/scene";
import { GridMaterial } from "@babylonjs/materials/grid/gridMaterial";
import { ref, onMounted, nextTick, watch } from "vue";

import * as BABYLON from "@babylonjs/core/Legacy/legacy";
const { pages } = defineProps({
  pages: Array,
});

const renderCanvas = ref(null);

const gizmoManager = ref(null);

const selectedSphereRef = ref(null);

const currentGizmoAction = ref("boundingBoxGizmoEnabled");
const toggleGizmo = (giz) => {
  if (gizmoManager.value) {
    gizmoManager.value[currentGizmoAction.value] = false;
    gizmoManager.value[giz] = !gizmoManager.value[giz];
  }
  currentGizmoAction.value = giz;
};
onMounted(() => {
  // Get the canvas element from the DOM.

  const canvas = renderCanvas.value;

  // Associate a Babylon Engine to it.
  const engine = new Engine(canvas);

  // Create our first scene.
  var scene = new Scene(engine);

  var camera = new BABYLON.FreeCamera(
    "camera1",
    new BABYLON.Vector3(10, 10, 10),
    scene
  );
  camera.setTarget(BABYLON.Vector3.Zero());
  // var camera = new BABYLON.FreeCamera(
  //   "camera1",
  //   new BABYLON.Vector3(0, 5, -5),
  //   scene
  // );
  // camera.setTarget(BABYLON.Vector3.Zero());
  // camera.attachControl(canvas, true);

  // const camera = new BABYLON.ArcRotateCamera(
  //   "camera-1",
  //   Math.PI - Math.PI / 4,
  //   Math.PI / 3,
  //   15,
  //   new BABYLON.Vector3(0, 0, 0),
  //   scene
  // );
  // camera.setTarget(new BABYLON.Vector3(0, 0, 0));
  // camera.attachControl(canvas);

  var light = new BABYLON.DirectionalLight(
    "light",
    new BABYLON.Vector3(0, -0.5, 1.0),
    scene
  );
  light.position = new BABYLON.Vector3(0, 5, -2);

  // Create simple meshes
  var spheres = [];

  for (var i = 0; i < 5; i++) {
    var sphere = BABYLON.Mesh.CreateSphere(`sphere${i}`, 16, 2, scene);
    sphere.material = new BABYLON.StandardMaterial("sphere material", scene);
    sphere.position.z = i;
    spheres.push(sphere);
    sphere.actionManager = new BABYLON.ActionManager(scene);
    sphere.material.diffuseColor = BABYLON.Color3.Blue();
    sphere.uniqField = "uniq";
    // sphere.actionManager.registerAction(
    //   new BABYLON.SetValueAction(
    //     {
    //       trigger: BABYLON.ActionManager.OnLeftPickTrigger,
    //     },
    //     sphere,
    //     "scaling",
    //     new BABYLON.Vector3(1.2, 1.2, 1.2)
    //   )
    // );

    // sphere.actionManager.registerAction(
    //   new BABYLON.ExecuteCodeAction(
    //     BABYLON.ActionManager.OnPickUpTrigger,
    //     function (evt) {
    //       evt.source.scaling = new BABYLON.Vector3(1, 1, 1);
    //     }
    //   )
    // );

    //     sphere.actionManager.registerAction(
    //   new BABYLON.ExecuteCodeAction(
    //     BABYLON.ActionManager.OnPointerOverTrigger,
    //     function (evt) {
    //       evt.source.scaling = new BABYLON.Vector3(1.2, 1.2, 1.2);
    //     }
    //   )
    // );

    // sphere.actionManager.registerAction(
    //   new BABYLON.ExecuteCodeAction(
    //     BABYLON.ActionManager.OnPointerOutTrigger,
    //     function (evt) {
    //       evt.source.scaling = new BABYLON.Vector3(1, 1, 1);
    //     }
    //   )
    // );
  }

  var selected = null;
  scene.onPointerObservable.add(function (evt) {
    if (selected) {
      selected.material.diffuseColor = BABYLON.Color3.Blue();
      selected = null;
    }
    console.log(evt.pickInfo.pickedMesh?.uniqField);
    if (
      evt.pickInfo.hit &&
      evt.pickInfo.pickedMesh &&
      evt.event.button === 0 &&
      evt.pickInfo.pickedMesh?.uniqField
    ) {
      selected = evt.pickInfo.pickedMesh;
      evt.pickInfo.pickedMesh.material.diffuseColor = BABYLON.Color3.Green();
    }
  }, BABYLON.PointerEventTypes.POINTERUP);

  // Initialize GizmoManager
  gizmoManager.value = new BABYLON.GizmoManager(scene);
  // gizmoManager.value.boundingBoxGizmoEnabled = true;
  // Restrict gizmos to only spheres
  gizmoManager.value.attachableMeshes = spheres;
  // Toggle gizmos with keyboard buttons
  document.onkeydown = (e) => {
    if (e.key == "w") {
      gizmoManager.value.positionGizmoEnabled =
        !gizmoManager.value.positionGizmoEnabled;
      console.log(gizmoManager.value.gizmos.positionGizmo);
    }
    if (e.key == "e") {
      gizmoManager.value.rotationGizmoEnabled =
        !gizmoManager.value.rotationGizmoEnabled;
    }
    if (e.key == "r") {
      gizmoManager.value.scaleGizmoEnabled =
        !gizmoManager.value.scaleGizmoEnabled;
    }
    if (e.key == "q") {
      gizmoManager.value.boundingBoxGizmoEnabled =
        !gizmoManager.value.boundingBoxGizmoEnabled;
    }
  };

  var ground = BABYLON.MeshBuilder.CreateGround(
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

  // Render every frame
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

#renderCanvas {
  width: 100%;
  height: 100%;
  display: block;
  font-size: 0;
}
</style>
