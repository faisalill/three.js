<script>
import { T, useFrame } from '@threlte/core';
import { OrbitControls } from '@threlte/extras';

let ref = null;

 useFrame((_, delta) => {
      if(ref) {
        ref.material.uniforms.uTime.value += delta;
      }
   })
</script>

<T.PerspectiveCamera position={[0, 0, 10]}>
  <OrbitControls enableDamping />
</T.PerspectiveCamera>

<T.Mesh
 bind:ref={ref}
>
  <T.PlaneGeometry args={[7, 7]} />
  <T.ShaderMaterial 
   uniforms={{
     uTime: { value: 0 }
   }}
   vertexShader={`
    uniform float uTime;
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
   `}
   fragmentShader={`
   uniform float uTime;
    varying vec2 vUv;
   void main() {
     vec2 changedUv = vUv - 0.5;
     changedUv.x += sin(vUv.y * 1.0 + uTime);
     changedUv.y += sin(vUv.x * 1.0 + uTime);
     changedUv *= vec2(5.0);
     changedUv = fract(changedUv);
      vec2 newUv = changedUv + vec2(sin(uTime) * 0.1, cos(uTime) * 0.1);
      vec3 variation = vec3(smoothstep(0.01, 0.9, length(newUv))); 

      vec3 color = vec3(1.0, 2.0, 3.0);
      color.b = sin(color.b * 10.0 + uTime);
      color.g = sin(color.g * 10.0 + uTime);

      vec3 final = variation;
      float numerator = 0.01;

      final = sin(numerator + uTime) / final;

      color *= final;

      gl_FragColor = vec4(color, 1.0);
   }
   `}
  />
</T.Mesh>
