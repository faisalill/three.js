<script>
  import { T } from '@threlte/core'
  import { OrbitControls } from '@threlte/extras';
</script>

<T.PerspectiveCamera
  position={[0, 0, 5]}
  makeDefault
>
  <OrbitControls />
</T.PerspectiveCamera>
<T.Mesh>
  <T.PlaneGeometry args={[4, 4, 4]} />
  <T.ShaderMaterial 
    vertexShader={`
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `}
    fragmentShader={`
      varying vec2 vUv;
      void main() {
        vec2 newUv = fract(vUv * 8.0);
        float size = 0.2;
        float variation = 1.0 - step(size, length(newUv - 0.5));
        gl_FragColor = vec4(vec3(variation), 1.0);
      }
    `}
  />
</T.Mesh>

