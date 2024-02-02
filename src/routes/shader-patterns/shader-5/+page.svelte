<script>
import * as THREE from 'three';
import  { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

let scene, camera, renderer;

onMount(() => {

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
  camera.position.z = 5;
  
  renderer = new THREE.WebGLRenderer();
  renderer.setSize( window.innerWidth, window.innerHeight );
  renderer.setClearColor( 0x000000, 1 );
  document.body.appendChild( renderer.domElement );

  const geometry = new THREE.PlaneGeometry( 5, 5 );
  const material = new THREE.ShaderMaterial({
    vertexShader: `
      varying vec2 vUv;
      void main() {
        gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1 );
        vUv = uv;
      }
    `,
    fragmentShader: `
      varying vec2 vUv;
      void main() {
        // float y = step(0.5, mod(vUv.y * 10.0, 1.0));
        float y = step(0.8, mod(vUv.y * 10.0, 1.0));
        float x = step(0.8, mod(vUv.x * 10.0, 1.0));
        // gl_FragColor = vec4(vec3(y), 1.0 );
        gl_FragColor = vec4(vec3(x + y), 1.0 );
      }
    `,
  });
  const plane = new THREE.Mesh( geometry, material );
  scene.add( plane );

  const controls = new OrbitControls( camera, renderer.domElement );
  controls.update()

  function animate() {
    requestAnimationFrame( animate );
    controls.update()
    renderer.render( scene, camera );
  }
  animate()
})

</script>
