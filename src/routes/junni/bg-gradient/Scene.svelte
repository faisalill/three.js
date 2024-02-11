<script>
  import { T, useFrame } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';

  let shaderMaterial;

  useFrame((_, delta) => {
    shaderMaterial.uniforms.time.value += delta;
  })
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 6]}> 
  <OrbitControls />
</T.PerspectiveCamera>

<T.Mesh
rotation.z={Math.PI / 4}
scale={2}
>
<T.PlaneGeometry args={[5, 5]} />
<T.ShaderMaterial 
  bind:ref={shaderMaterial}
  uniforms={{
    time: { value: 0 }
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
    vec3 hsv2rgb( vec3 hsv ){

      return ((clamp(abs(fract(hsv.x+vec3(0,2,1)/3.)*6.-3.)-1.,0.,1.)-1.)*hsv.y+1.)*hsv.z;

    }
    varying vec2 vUv;
    uniform float time;
    void main() {
      vec3 color = hsv2rgb( vec3( -vUv.x * 0.2 + 0.3 + time * 0.1 + random( gl_FragCoord.xy * 0.01 ) * 0.02, 0.95, 1.0  ) );
      gl_FragColor = vec4(color, 1.0);
    }
  `}
  />
</T.Mesh>  
