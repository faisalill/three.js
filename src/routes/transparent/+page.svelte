<script>
import * as THREE from 'three';
import { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

let camera, scene, renderer;
let geometry, material, mesh;

onMount(() => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 1000 ); 
  camera.position.z = 8;
  camera.position.x = 4;
  camera.position.y = 4;

  geometry = new THREE.BoxGeometry( 2, 2, 2 );
  const planeGeometry = new THREE.PlaneGeometry( 5, 5, 32 );

  const basicMaterial = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
  material = new THREE.ShaderMaterial({
    vertexShader: `
      void main() {
        gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
      }
    `,
    fragmentShader: `
      void main() {
        gl_FragColor = vec4(1.0, 0.0, 0.0, 0.5);
      }
    `,
    transparent: true,
  });

  mesh = new THREE.Mesh( geometry, basicMaterial );
  scene.add( mesh );

  const secondMesh = new THREE.Mesh( planeGeometry, material );
  secondMesh.position.z = 2;
  scene.add( secondMesh );

  renderer = new THREE.WebGLRenderer( { antialias: true, alpha: true } );
  renderer.setSize( window.innerWidth, window.innerHeight );
  renderer.setClearColor( 0x000000);
  document.body.appendChild( renderer.domElement );

  const controls = new OrbitControls( camera, renderer.domElement );
  controls.update();

  function animate() {
    requestAnimationFrame( animate );
    controls.update();
    renderer.render( scene, camera );
  }

  animate();
    
})
</script>



