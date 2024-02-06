<script>
import * as THREE from 'three';
import { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

let scene, camera, renderer;

onMount(() => {
  scene = new THREE.Scene();

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 25;
  camera.position.y = 25;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.outputEncoding = THREE.sRGBEncoding;
  document.body.appendChild(renderer.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.7/');
  loader.setDRACOLoader(dracoLoader);

  const bakedTexture = new THREE.TextureLoader().load('/baking.jpg');
  bakedTexture.flipY = false;
  bakedTexture.encoding = THREE.sRGBEncoding;

  loader.load('/house.glb', (gltf) => {
    gltf.scene.traverse((child) => {
      if(child.isGroup) {
        child.children.forEach((mesh) => {
          if(mesh.isMesh) {
            mesh.material = new THREE.MeshBasicMaterial({ map: bakedTexture });
          }
        });
      }
      if(child.isMesh) {
        child.material = new THREE.MeshBasicMaterial({ map: bakedTexture });
      }
    });
    scene.add(gltf.scene);
  });


  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
}

  animate();
})


</script>
