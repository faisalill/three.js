<script>
  import { T, useFrame, useThrelte } from '@threlte/core';
  import { useTexture, OrbitControls } from '@threlte/extras';
  import * as motion from 'motion'
  import { onMount } from 'svelte';

  const firstText = useTexture('/flexible.png');
  const secondText = useTexture('/bg-text.png');
  let shaderMaterialRef;
  let cameraPosition ={
    x: 0, 
    y: 0, 
    z: 4
  };
  let cameraRef;

  const { scene } = useThrelte();

  onMount(( ) => {
    window.addEventListener(('mousemove'), (e) => {
      motion.animate((progress) => {
        cameraPosition.x = 0.2 * (e.clientX - window.innerWidth / 2) / 100;
      },
      {
          easing: 'easeInOut',
        }
      )
      motion.animate((progress) => {
        cameraPosition.y = 0.2 * -(e.clientY - window.innerHeight / 2) / 100;
      },
      {
          easing: 'easeInOut',
        }
      )
      cameraRef.lookAt(scene.position);
    })
  })

  useFrame((_, delta) => {

    if (shaderMaterialRef) {
      shaderMaterialRef.uniforms.uTime.value += delta;
    }
  })
</script>

<T.PerspectiveCamera 
  makeDefault 
  position={[cameraPosition.x, cameraPosition.y, cameraPosition.z]} 
  bind:ref={cameraRef}
>
  <!-- <OrbitControls /> -->
</T.PerspectiveCamera>

<T.Mesh position={[0, 0, -4]} scale={2}>
  <T.PlaneGeometry
    args={[5,5]}
    />
  <T.MeshBasicMaterial
    color={'#E8E8E8'} 
    />
</T.Mesh>

{#await secondText then value}
  <T.Mesh position={[0, 0, -2]} scale={1.5} >
    <T.PlaneGeometry
      args={[5,5]}
      />
    <T.ShaderMaterial
      bind:ref={shaderMaterialRef}
      transparent
      uniforms={{
        uTexture: { value: value },
        uTime: { value: 0.0 }
      }}
      vertexShader={`
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `}
      fragmentShader={`
        varying vec2 vUv;
        uniform sampler2D uTexture;
        uniform float uTime;
        void main() {
 // Determine the row index based on vUv.y
    float rowIndex = floor(vUv.y * 9.0);
    
    // Adjust UV coordinates to stretch the texture vertically
    vec2 newUv = vUv * vec2(1.0, 9.0);
    
    // Apply horizontal movement based on vertical position and rowIndex
    newUv.x += vUv.y * 0.2 * (1.0 + rowIndex * 0.1); // Adjust the multiplier as needed

    // Vertical Skew
    newUv.y -= vUv.x * 2.7;
    
    // Wrap UV coordinates to simulate infinite repeating
    newUv = fract(newUv + vec2(uTime * 0.2 * (1.0 + rowIndex * 0.05), 0.0)); // Adjust the multiplier as needed
    

    vec4 text = texture2D(uTexture, newUv);
    text.rgb -= 0.5;

    // Sample the texture using the modified UV coordinates
    gl_FragColor = text;
        }
      `}
    />
  </T.Mesh>
{/await}

{#await firstText then value}
  <T.Mesh >
    <T.PlaneGeometry 
      args={[5,5]}
      />
    <T.ShaderMaterial
      transparent
      uniforms={
          {
            uTexture: { value: value },
          }
        }
      vertexShader={`
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `}
      fragmentShader={`
        varying vec2 vUv;
        uniform sampler2D uTexture;
        void main() {
          vec2 newUv = vUv;
          vec4 text = texture2D(uTexture, newUv); 
          float darkeningFactor = 1.5;
          text.w *= darkeningFactor;
          gl_FragColor = text;
        }
      `}
      />

  </T.Mesh>
{/await}
