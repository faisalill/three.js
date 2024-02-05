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
  camera.position.z = 8.5;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000, 1);
  document.body.appendChild(renderer.domElement);

  const geometry = new THREE.PlaneGeometry(10, 10, 20, 20);
  const material = new THREE.ShaderMaterial({
    transparent: true,
    uniforms: {
      time: { value: 0 },
      speed: { value: 1.0 },
      hollows: { value: 10.0 },
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
          vec3 a = vec3( 0.668, 0.668, 0.668 );
          vec3 b = vec3( 0.5, 0.5, 0.5 );
          vec3 c = vec3( 1.0, 1.0, 1.0 );
          vec3 d = vec3( 0.000, 0.333, 0.667 );
          return a + b*cos( 6.28318*(c*t+d) );
      }
      varying vec2 vUv;
      uniform float time;
      uniform float speed;
      uniform float hollows;
      uniform float thickness;

      void main() {
        vec2 uv = vUv;
        vec2 uv0 = uv;
        vec3 finalColor = vec3(0.0);
    
        for (float i = 0.0; i < 3.0; i++) {

        uv = fract(uv * 2.0) - 0.5;

        float d = length(uv);
        
        vec3 col = palette(length(uv0 - 0.5) * 8.0 + time * 0.5 * speed + i * 2.0);
  
        d = sin(d * hollows  + time)/8.0;
        d = abs(d);
        d = pow(0.02/d, 1.2);
        
        finalColor += col * d;        
        } 

        gl_FragColor = vec4(finalColor, 1.0);
      }
    `
  })

  const plane = new THREE.Mesh(geometry, material);
  scene.add(plane);

  gui.add(material.uniforms.speed, 'value', 1, 10).name('Speed');
  gui.add(material.uniforms.hollows, 'value', 6, 20).name('Hollows');
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
