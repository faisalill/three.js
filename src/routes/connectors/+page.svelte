<script>
import * as THREE from 'three'
import { onMount } from 'svelte'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

let scene, camera, renderer;

onMount(() => {
  scene = new THREE.Scene();

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 5;
  camera.position.y = 5;

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const material = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 1.0 }
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
      uniform float time;
      void main() {
        
        gl_FragColor = vec4(sin(gl_FragCoord + time * 8.0));
      }
    `
  });

  const points = [];
  points.push( new THREE.Vector3( - 5, 0, 0 ) );
  points.push( new THREE.Vector3( 5, 0, 0 ) );

  const geometry = new THREE.BufferGeometry().setFromPoints(points);

  const line = new THREE.Line( geometry, material );
  scene.add( line );

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  const clock = new THREE.Clock();
  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    material.uniforms.time.value = clock.getElapsedTime();
    renderer.render(scene, camera);
  }

  animate();
})
</script>
