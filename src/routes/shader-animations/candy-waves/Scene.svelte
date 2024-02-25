<script>
import { T, useFrame } from '@threlte/core';
import { OrbitControls } from '@threlte/extras';
import { DoubleSide } from 'three';

let eyeCandyBackground;

useFrame((_, delta) => {
  if (eyeCandyBackground) {
    eyeCandyBackground.material.uniforms.uTime.value += delta;
  }
});
</script>

<T.PerspectiveCamera makeDefault position={[10, 0, 0]} >
  <OrbitControls enableDamping/>
</T.PerspectiveCamera>


<T.Mesh
position={[-4, 0, 0]}
rotation={[ 0, Math.PI / 2, 0]}
bind:ref={eyeCandyBackground}
>
  <T.PlaneGeometry args={[15, 15]} />
  <T.ShaderMaterial 
  transparent={true}
  side={DoubleSide}
  uniforms={{
    uTime: { value: 0 }
  }}
  vertexShader={` 
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `}
  fragmentShader={` 
    float random(vec2 p){
      return fract(sin(dot(p.xy ,vec2(12.9898,78.233))) * 43758.5453);
    }
    varying vec2 vUv;
    uniform float uTime;
    void main() { 
      vec2 newUv = vUv;
      vec3 color = vec3(1.0, 2.0, 3.0);
      color.r = sin(uTime + length(vUv) * 90.0) * 0.5 + 0.5;
      color.g = cos(uTime + length(vUv) * 90.0) * 0.5 + 0.5;

      vec3 final = vec3(length(newUv - 0.5));
      final = 0.11 / final;
      final = smoothstep(0.25, 0.30, final);
      color *= final;
      gl_FragColor = vec4(color, 1.0 - length(vUv - 0.5));
    }
  `}
  />
</T.Mesh>
