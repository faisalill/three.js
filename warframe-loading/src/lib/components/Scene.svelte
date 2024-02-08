<script>
  import { T, useThrelte, useRender } from '@threlte/core';
  import { ContactShadows, Float, Grid, OrbitControls } from '@threlte/extras';
  import SpaceShip from '$lib/components/models/space_ship.svelte';
  import Stars from './Stars.svelte';
  import { Vector3, Raycaster, Vector2, PMREMGenerator, Color } from 'three';
  import { onMount } from 'svelte';
  import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
  import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
	import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
	import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';

  let cameraPosition = new Vector3(-7.41306, 7.61660, 12.7292);
  let cameraTarget = new Vector3(0, 0, 0);
  let sphereRef, intersectionPoint, planeRef, spaceShipRef, environmentMap;
  let translateY = 0;
  let translateAcceleration = 0;
  let angle = 0;
  let angelAcceleration = 0;

  let { camera, renderer, scene } = useThrelte();
  let composer = new EffectComposer(renderer);

  let pmremGenerator = new PMREMGenerator(renderer);

  
  useRender(() => {
    if (intersectionPoint) {
      intersectionPoint.x = -6;
      const targetY = intersectionPoint?.y;
      translateAcceleration += (targetY - translateY) * 0.005;
      translateAcceleration *= 0.95;
      translateY += translateAcceleration;

      const direction = intersectionPoint.clone().sub(new Vector3(0, translateY, 0)).normalize();
      const directionCos = direction.dot(new Vector3(0, 1, 0));
      const targetAngle = Math.acos(directionCos) - Math.PI / 2;
      angelAcceleration += (targetAngle - angle) * 0.005;
      angelAcceleration *= 0.95;
      angle += angelAcceleration;
    }

    if(environmentMap) {
      environmentMap.dispose()
    }

    spaceShipRef.visible = false;
    scene.background = null;
    environmentMap = pmremGenerator.fromScene(scene, 0, 0.1, 1000);
    scene.background = new Color('#598889').multiplyScalar(0.05);
    spaceShipRef.visible = true;

    spaceShipRef.traverse((child) => {
      if (child?.material?.envMapIntensity){
        child.material.envMap = environmentMap.texture;
        child.material.envMapIntensity = 100;
        child.material.normalScale.set(0.3, 0.3);
      }
    })

    composer.render();
  })

  onMount(async() => {
    const dat = await import('dat.gui'); 
    const gui = new dat.GUI();

    let params = {
      bloomStrength: 0.3,
      bloomThreshold: 0,
      bloomRadius: 0
    };

    composer.setSize(window.innerWidth, window.innerHeight);
    
    let renderPass = new RenderPass(scene, camera.current);
    composer.addPass(renderPass);

    let bloomPass = new UnrealBloomPass(new Vector2(window.innerWidth, window.innerHeight), params.bloomStrength, params.bloomThreshold, params.bloomRadius);
  
    gui.add(params, 'bloomStrength', 0.0, 10.0).onChange(function (value) {
      bloomPass.strength = Number(value);
    });

    gui.add(params, 'bloomThreshold', 0.0, 10.0).onChange(function (value) {
      bloomPass.threshold = Number(value);
    });

    gui.add(params, 'bloomRadius', 0.0, 10.0).onChange(function (value) {
      bloomPass.radius = Number(value);
    });
    composer.addPass(bloomPass);

    let outputPass = new OutputPass();
    composer.addPass(outputPass);

    function onPointerMove (event) {
      const pointer = new Vector2();      
      pointer.x = ( event.clientX / window.innerWidth ) * 2 - 1;
      pointer.y = - ( event.clientY / window.innerHeight ) * 2 + 1;

      const raycaster = new Raycaster();
      raycaster.setFromCamera( pointer, $camera );
      
      const intersects = raycaster.intersectObject( planeRef ); 
      if (intersects.length > 0)
        intersectionPoint = intersects[0]?.point;
        sphereRef.position.copy(intersects[0].point); 
      }
      window.addEventListener('pointermove', onPointerMove)
      return () => {
        window.removeEventListener('pointermove', onPointerMove) 
      }
  }) 
  </script>

<T.PerspectiveCamera position={[cameraPosition.x, cameraPosition.y, cameraPosition.z]} makeDefault
>
  <OrbitControls 
    enableDamping
  />
</T.PerspectiveCamera>

<T.DirectionalLight intensity={2} position={[0, 15, 0]} castShadow shadow.bias={-0.0001}/>
<T.AmbientLight intensity={2} />

<Grid
  visible={false}
  fadeDistance={60}
  sectionThickness={0}
  cellColor={0xfffffff}
/>

<T.Mesh position.z={-4} bind:ref={planeRef} visible={false}>
<T.PlaneGeometry 
    args={[200, 40]} 
  />

<T.MeshStandardMaterial 
    color={'#dd9d31'} 
    transparent
    opacity={0.35}
  />
</T.Mesh>

<T.Mesh bind:ref={sphereRef} visible={false}>
  <T.SphereGeometry 
    args={[0.5, 32, 32]} />
  <T.MeshStandardMaterial color={'#ff0000'} />
</T.Mesh>
  
<Stars />

<SpaceShip bind:ref={spaceShipRef} scale={0.8} position={[0, translateY, 0]} rotation={[angle, 0, angle, 'ZXY']}>

</SpaceShip>
