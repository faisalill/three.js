<script>
import { onMount } from 'svelte';
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

let scene, camera, renderer;

onMount(async () => {
  const dat = await import('dat.gui');
  const gui = new dat.GUI();

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 3;
  camera.position.y = 15;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const geometry = new THREE.BoxGeometry(150, 150);
  const material = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 1.0 },
      resolution: { value: new THREE.Vector2(window.innerWidth, window.innerHeight) },
      spread: { value: 0.011 },
      speed: { value: 0.5}
    },
    vertexShader: `
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      uniform float time;
      uniform vec2 resolution;
      uniform float spread;
      uniform float speed;
      #define PI 3.14159265359

      void main(void) {
        vec3 redColor = vec3(0.9, 0.0, 0.3);
        vec3 greenColor = vec3(0.0, 0.9, 0.3);
        vec3 blueColor = vec3(0.0, 0.3, 0.9);
        vec3 yelloColor = vec3(0.9, 0.9, 0.3);

        vec2 position = (gl_FragCoord.xy * 2.0 - resolution);
        position /= min(resolution.x, resolution.y);

        float a = sin(position.y * 5.0 - time * speed) /2.0;
        float b = cos(position.y * 5.0 - time * speed) /2.0;
        float c = sin(position.y * 5.0 - time * speed + PI) /2.0;
        float d = cos(position.y * 5.0 - time * speed + PI) /2.0;
        float colorSpread = spread; 
// Make colorspread vary along with time
        float e = colorSpread / abs(position.x + a);
        float f = colorSpread / abs(position.x + b);
        float g = colorSpread / abs(position.x + c);
        float h = colorSpread / abs(position.x + d);

        vec3 destinationColor = redColor * e + greenColor * f + blueColor * g + yelloColor * h;

        gl_FragColor = vec4(destinationColor, 1.0);
      }
    `
  });

  gui.add(material.uniforms.spread, 'value', 0.001, 0.4).name('Spread');

  gui.add(material.uniforms.speed, 'value', 0.1, 2).name('Speed');
  
  const plane = new THREE.Mesh(geometry, material);
  plane.rotation.x = Math.PI / 2;
  scene.add(plane);

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
