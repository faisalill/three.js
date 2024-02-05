<script>
import { onMount } from 'svelte';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
let scene, camera, renderer;

onMount(async() => {
  const dat = await import('dat.gui');
  const gui = new dat.GUI();

  scene = new THREE.Scene();

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 6;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000, 1);
  document.body.appendChild(renderer.domElement);

  const geometry = new THREE.PlaneGeometry(10, 10, 20, 20);
  const material = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      speed: { value: 29.8 },
      hollows: { value: 92.0 },
      thickness: { value: 61.0 }
    },
    vertexShader: `
      uniform float time;
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      varying vec2 vUv;
      uniform float time;
      uniform float speed;
      uniform float hollows;
      uniform float thickness;

      void main() {
        float color = length(vUv - 0.5);
        color = sin(color * hollows + time * speed) / thickness;
        color = abs(color);
        color = 1.0 - smoothstep(0.006,0.011, color);
        gl_FragColor = vec4(vec3(color), 1.0);
      }
    `
  })
  const plane = new THREE.Mesh(geometry, material);
  scene.add(plane);

  gui.add(material.uniforms.speed, 'value', 1, 100).name('Speed');
  gui.add(material.uniforms.hollows, 'value', 10, 200).name('Hollows');
  gui.add(material.uniforms.thickness, 'value', 10, 200).name('Thickness');

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  const clock = new THREE.Clock();
  function animate() {
    requestAnimationFrame(animate);
    material.uniforms.time.value = clock.getElapsedTime();
    controls.update();
    renderer.render(scene, camera);
  }

  animate()
}
)

</script>
