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

  const texturePass = {
    uniforms: {
      tDiffuse: { value: null },
      uNormalMap: { value: null },
      uTime: { value: 0 },
    },
    vertexShader: `
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      uniform sampler2D tDiffuse;
      uniform sampler2D uNormalMap;
      uniform float uTime;
      varying vec2 vUv;
      void main() {
        vec4 normal = texture2D(uNormalMap, vUv);
        vec2 newvUv = vUv + normal.xy * sin(uTime) / 2.0;
        vec4 color = texture2D(tDiffuse, newvUv);
        gl_FragColor = color;
      }
    `,
  };

  const shaderPass = new ShaderPass(texturePass);
  shaderPass.material.uniforms.uNormalMap.value = new THREE.TextureLoader().load('/normal_map.png');
  composer.addPass(shaderPass);

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
    shaderPass.material.uniforms.uTime.value = clock.getElapsedTime();
    // renderer.render(scene, camera);
    composer.render();
  };

  animate();
});


</script>

