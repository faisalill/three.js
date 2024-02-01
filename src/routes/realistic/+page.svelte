<script>
import * as THREE  from 'three';
import { onMount } from 'svelte';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js';

let scene, camera, renderer;
let model;

onMount(()=>{

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

  camera.position.x = 1;
  camera.position.y = 1.5;
  camera.position.z = 2;

  window.addEventListener( 'keydown', (e) =>{
    // change the model position using WSAD
    if (e.key === 'w') {
      model.position.z += 0.1;
    }
    if (e.key === 's') {
      model.position.z -= 0.1;
    }
    if (e.key === 'a') {
      model.position.x += 0.1;
    }
    if (e.key === 'd') {
      model.position.x -= 0.1;
    }

  })

  renderer = new THREE.WebGLRenderer();
  renderer.setSize( window.innerWidth, window.innerHeight );
  renderer.setClearColor( 0x000000, 1 );
  renderer.physicallyCorrectLights = true;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.gammaFactor = 0.2;
  // renderer.toneMapping = THREE.ACESFilmicToneMapping;
  // renderer.toneMappingExposure = 0.5;
  document.body.appendChild( renderer.domElement );

  const gridHelper = new THREE.GridHelper( 10, 10 );
  scene.add( gridHelper );

  const axesHelper = new THREE.AxesHelper( 50 );
  scene.add( axesHelper );

  const controls = new OrbitControls( camera, renderer.domElement );
  controls.update();

  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath( 'https://www.gstatic.com/draco/versioned/decoders/1.5.7/' );
  loader.setDRACOLoader( dracoLoader );

  loader.load( '/blossom_box.glb', function ( gltf ) {
    model = gltf.scene;
    scene.add( model );
    const middleColor = new THREE.Color('#000000')
    let finalColor = new THREE.Color(colorstr)
    finalColor = finalColor.convertSRGBToLinear()

    model.traverse((node) => {
        node.material.color.copy(middleColor).lerp(finalColor, op2)
    })
  }, undefined, function ( error ) {
    console.error( error );
  } );

  const rgbeLoader = new RGBELoader();
  rgbeLoader.load('/studio.hdr', function (texture) {
    // texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.environment = texture;
  })

  // const hemiLight = new THREE.HemisphereLight( 0xffffbb, 0x080820, 45 );
  // scene.add( hemiLight )
  //
  // const ambientLight = new THREE.AmbientLight( 0xffffff, 10 );
  // scene.add( ambientLight );

  function animate() {
    requestAnimationFrame( animate );

    controls.update();

    renderer.render( scene, camera );
  }
  animate();
})
</script>

