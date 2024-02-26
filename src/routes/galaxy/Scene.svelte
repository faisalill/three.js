<script>
import { T } from '@threlte/core';
import { OrbitControls, useTexture, Instance, InstancedMesh } from '@threlte/extras';
import { ShaderMaterial, Sprite } from 'three';

const starSize = 0.5;
const starCount = 100;

const sphereTexture = useTexture('/sphere.png');
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 40]}>
  <OrbitControls enableDamping />
</T.PerspectiveCamera>
  
{#await sphereTexture then texture}
  <InstancedMesh>
    <T.PlaneGeometry args={[starSize, starSize]} />
    <T.ShaderMaterial 
      uniforms={{ 
        map: { value: texture }
      }}
      vertexShader={`
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `}
      fragmentShader={`
        uniform sampler2D map;
        varying vec2 vUv;
        void main() {
          gl_FragColor = texture2D(map, vUv);
        }
      `}
    />
      
    <Instance />
  </InstancedMesh>
{/await}

