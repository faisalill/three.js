<script>
import * as THREE from 'three'
import { onMount } from 'svelte'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

let scene, camera, renderer;

onMount(() => {
  scene = new THREE.Scene();

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 25;

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const geometry = new THREE.IcosahedronGeometry(10, 2);
  const material = new THREE.ShaderMaterial({ 
    wireframe: true,
    uniforms: {
      time: { value: 1.0 }
    },
    vertexShader: `
      uniform float time;
      varying vec2 vUv; 
      void main() {
        vUv = uv;
        vec3 newPosition = position;
        newPosition.y += sin(position.x + time) * 0.4;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition , 1.0);
        // gl_Position = projectionMatrix * modelViewMatrix * vec4(position , 1.0);
      }
    `,
    fragmentShader: `
      uniform float time;
      varying vec2 vUv; 
      void main() {
        vec3 color = vec3(1.0, 2.0, 3.0); 
        float variation = vUv.y + sin(time) * 0.1;

        gl_FragColor = vec4(color * variation, 1.0);
      }
    `
  });

  const mesh = new THREE.Mesh(geometry, material);
  scene.add(mesh);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  const clock = new THREE.Clock();
  function animate() {
    requestAnimationFrame(animate);
    material.uniforms.time.value = clock.getElapsedTime();
    controls.update();
    renderer.render(scene, camera);
  }

  animate();
})


</script>
