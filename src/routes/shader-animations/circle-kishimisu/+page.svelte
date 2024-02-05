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
  camera.position.z = 6.5;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000, 1);
  document.body.appendChild(renderer.domElement);

  const geometry = new THREE.PlaneGeometry(10, 10, 20, 20);
  const material = new THREE.ShaderMaterial({
    transparent: true,
    uniforms: {
      time: { value: 0 },
      speed: { value: 2.0 },
      hollows: { value: 20.0 },
      thickness: { value: 94.0 }
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
      vec3 palette( in float t )
      {
          vec3 a = vec3( 0.5, 0.5, 0.5 );
          vec3 b = vec3( 0.5, 0.5, 0.5 );
          vec3 c = vec3( 1.0, 1.0, 1.0 );
          vec3 d = vec3( 0.263, 0.416, 0.557 );
          return a + b*cos( 6.28318*(c*t+d) );
      }
      varying vec2 vUv;
      uniform float time;
      uniform float speed;
      uniform float hollows;
      uniform float thickness;

      void main() {
        
        vec2 newvUv = fract(vUv);
        float variation = length(newvUv - 0.5);
        
        vec3 color1 = palette(length(vUv - 0.5) * 10.0 + time / 1.5);

        variation = sin(variation * hollows + time * speed) / thickness;
        variation = abs(variation);
        variation = 1.0 - smoothstep(0.006,0.011, variation);
        variation = 0.21 / variation;

        vec3 finalValue = vec3(color1 * vec3(variation));
        
        gl_FragColor = vec4(finalValue, 1.0);
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
