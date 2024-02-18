<script>
 import { T, useFrame } from '@threlte/core';
 import { OrbitControls, useTexture } from '@threlte/extras';

 let materialRef; 
 let randomTexture = useTexture('/random.png');
 let displayTexture = useTexture('/display.png');
 let rgbNoiseTexture = useTexture('/noise.png');

 useFrame((_, delta) => {
    if (materialRef) {
      materialRef.uniforms.uTime.value += delta;
    }
 });

</script>


<T.PerspectiveCamera makeDefault position={[0, 0, 11]}>
  <OrbitControls />
</T.PerspectiveCamera>

{#await randomTexture then random}
  {#await displayTexture then display}
    {#await rgbNoiseTexture then rgbNoise}
  <T.Mesh>
    <T.PlaneGeometry args={[9, 9]} />
    <T.ShaderMaterial 
      bind:ref={materialRef}
      uniforms={{
        uTime: { value: 0 },
        uRandomTexture: { value: random },
        uDisplayTexture: { value: display },
        uRgbNoiseTexture: { value: rgbNoise }
      }}
      vertexShader={`
        uniform float uTime;
        varying vec2 vUv;
        void main() {
          vUv = uv;
          vec3 newPos = position;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(newPos, 1.0);
        }
      `}
      fragmentShader={`
        uniform float uTime;
        uniform sampler2D uRandomTexture;
        uniform sampler2D uDisplayTexture;
        uniform sampler2D uRgbNoiseTexture;
        varying vec2 vUv;
        void main() {
          vec2 newUv = vUv;
          newUv.x += sin(uTime * 100.0);
          vec4 randomColor = texture2D(uRandomTexture, newUv); 
          vec4 displayColor = texture2D(uDisplayTexture, vUv);
          gl_FragColor = randomColor;
          // gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
        }
      `}
    />
  </T.Mesh>
    {/await}
  {/await}
{/await}
