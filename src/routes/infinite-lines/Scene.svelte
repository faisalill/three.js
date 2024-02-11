<script>
  import { T, useFrame } from '@threlte/core';
  import { OrbitControls, TransformControls } from '@threlte/extras';
  import { DoubleSide, Clock } from 'three';

  let shaderMaterial;

  useFrame((_, delta) => {
    if(shaderMaterial) {
      shaderMaterial.uniforms.time.value += delta;
    }
  });

</script>

<T.AmbientLight intensity={0.5} />

<T.PerspectiveCamera makeDefault position={[10, 10, 10]}>
  <OrbitControls />
</T.PerspectiveCamera>

<T.Mesh 
  rotation.x={Math.PI/2}
  rotation.z={Math.PI/4 * 0.1}
  position={[-2, 0, 0]}
>
  <T.PlaneGeometry
    args={[20, 4]}
  />
  <T.ShaderMaterial 
    bind:ref={shaderMaterial}
    uniforms={
      {
        time: { value: 0.0}
      }
    }
    transparent
    side={DoubleSide}
    vertexShader={`
      varying vec2 vUv;
      uniform float time;
      void main() {
        vUv = uv;
        vec3 pos = position;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
      }
    `}
    fragmentShader={`
      varying vec2 vUv;
      uniform float time;
      void main() {
        vec2 newUv = vUv;
        newUv.y -= 0.2;
        newUv.y += sin(newUv.x * 10.0 + time * 2.0) * 0.1;
        newUv.y = fract(newUv.y * 10.0);
        gl_FragColor = vec4(vec3( step(0.5, sin(newUv.y * 5.0)) ),vUv.x + 1.0);
      }
    `}
  />
</T.Mesh>
