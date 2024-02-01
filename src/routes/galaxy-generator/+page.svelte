<script>
import * as THREE from 'three';
import { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

let camera, scene, renderer;
let geometry, material;

onMount(async()=>{
  const dat = await import('dat.gui')
  const gui = new dat.GUI({
    width: 300,
  })

  let guiParams = {
    count: 3000,
    size: 0.03,
    camera: {
      x: 0,
      y: 0,
      z: 0
    },
    radius: 10,
    branches: 3,
  } 

  gui.add(guiParams, 'count').min(100).max(10000).step(100).onChange(()=>{
    drawGalaxy(guiParams.count, geometry)
  })

  gui.add(guiParams, 'radius').min(0.1).max(20).step(0.1).onChange(()=>{
    drawGalaxy(guiParams.count, geometry)
  })

  function drawGalaxy(count, geometry, radius = guiParams.radius) {
    const positions = new Float32Array( count * 3 );

    for ( let i = 0; i < count; i ++ ) {
      const i3 = i * 3;
      
     const branchAngle = (i % guiParams.branches) / guiParams.branches * Math.PI * 2

      positions[ i3 ] = ( Math.random()  * radius) * 10;
      positions[ i3 + 1 ] = 0;
      positions[ i3 + 2] = positions[ i3 ] * radius;
      // positions[ i + 1] = ( Math.random() - 0.5 ) * 10;
      // positions[ i + 2] = ( Math.random() - 0.5 ) * 10;
    }

    geometry.setAttribute( 'position', new THREE.BufferAttribute( positions, 3 ) );
  }

  gui.add(guiParams, 'size').min(0.001).max(0.5).step(0.001).onChange(()=>{
    material.size = guiParams.size
    material.needsUpdate = true
  })

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
  camera.position.z = 2;
  camera.position.y = 2;
  camera.position.x = 25;

  renderer = new THREE.WebGLRenderer();
  renderer.setSize( window.innerWidth, window.innerHeight );
  document.body.appendChild( renderer.domElement );

  const controls = new OrbitControls( camera, renderer.domElement );
  controls.update();

  geometry = new THREE.BufferGeometry();
  drawGalaxy(guiParams.count, geometry);

  material = new THREE.PointsMaterial( {
    size: guiParams.size,
    sizeAttenuation: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
  } );
  const points = new THREE.Points( geometry, material );
  scene.add( points );

  function animate() {
    requestAnimationFrame( animate );
    controls.update();
    renderer.render( scene, camera );
  }

  animate();
})

</script>
