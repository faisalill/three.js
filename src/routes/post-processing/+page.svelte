<script>
import * as THREE from 'three';
import { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
import { GlitchPass } from 'three/examples/jsm/postprocessing/GlitchPass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';

let scene, camera, renderer, model;

onMount(async () => {
  const dat = await import('dat.gui')
  const gui = new dat.GUI({
  })

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 2;
  camera.position.x = 2.5;
  camera.position.y = 2;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const composer = new EffectComposer(renderer);
  composer.addPass(new RenderPass(scene, camera));

  const tintShader = {
    uniforms: {
      tDiffuse: { value: null },
      uTint: { value: null}
    },
    vertexShader: `
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      uniform vec3 uTint;
      varying vec2 vUv;
      uniform sampler2D tDiffuse;
      void main() {
        vec4 color = texture2D(tDiffuse, vUv);
        color.rgb += uTint;
        gl_FragColor = vec4(color.r + 0.2, color.g + 0.2, color.b + 0.2, color.a);
      }
    `
  }

  const tintPass = new ShaderPass(tintShader);
  tintPass.uniforms.uTint.value = new THREE.Vector3(0.0, 0.0, 0.0);
  composer.addPass(tintPass);

  gui.add(tintPass.uniforms.uTint.value, 'x').min(0).max(1).step(0.01).name('red').onChange(() => {
    tintPass.uniforms.uTint.value.x = tintPass.uniforms.uTint.value.x;
  });
  gui.add(tintPass.uniforms.uTint.value, 'y').min(0).max(1).step(0.01).name('green').onChange(() => {
    tintPass.uniforms.uTint.value.y = tintPass.uniforms.uTint.value.y;
  });
  gui.add(tintPass.uniforms.uTint.value, 'z').min(0).max(1).step(0.01).name('blue').onChange(() => {
    tintPass.uniforms.uTint.value.z = tintPass.uniforms.uTint.value.z;
  });

  const displacementShader = {
    uniforms: {
      tDiffuse: { value: null },
      uTime: { value: 0 },
      uHeight: { value: 8.0 },
      uWavy: { value: 1.0 }
    },
    vertexShader: `
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      varying vec2 vUv;
      uniform float uTime;
      uniform float uHeight;
      uniform float uWavy;
      uniform sampler2D tDiffuse;
      void main() {
        vec2 newUv = vec2(
        vUv.x,
        vUv.y
);
        newUv.y += sin(vUv.x * uWavy + (uTime) ) / uHeight;;  
        vec4 color = texture2D(tDiffuse, newUv);
        gl_FragColor = color;
      }
    `
  }

  const displacementPass = new ShaderPass(displacementShader);
  composer.addPass(displacementPass);

  gui.add(displacementPass.uniforms.uHeight, 'value').min(0).max(20).step(0.01).name('height').onChange(() => {
    displacementPass.uniforms.uHeight.value = displacementPass.uniforms.uHeight.value;
  });
  gui.add(displacementPass.uniforms.uWavy, 'value').min(0).max(50).step(0.01).name('wavyness').onChange(() => {
    displacementPass.uniforms.uWavy.value = displacementPass.uniforms.uWavy.value;
  });

  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.7/');
  loader.setDRACOLoader(dracoLoader);
  loader.load('/blossom_box.glb', (gltf) => {
    scene.add(gltf.scene);
  });

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  const clock = new THREE.Clock();
  const animate = () => {
    requestAnimationFrame(animate);
    controls.update();
    displacementPass.uniforms.uTime.value = clock.getElapsedTime();
    // renderer.render(scene, camera);
    composer.render();
  };

  animate();
});


</script>

