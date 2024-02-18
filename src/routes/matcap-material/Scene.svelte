<script>
import { T, useFrame } from '@threlte/core';
import { OrbitControls, useTexture } from '@threlte/extras';

let icoRef, icoMaterialRef;
const silverMatcap = useTexture('/matcap-silver.jpg')
const rgbMatcap = useTexture('/matcap-rgb.png') 
const pinkReflectionMatcap = useTexture('/matcap-pink-reflection.png')
const transparentReflectionMatcap = useTexture('/matcap-transparent.png')
const toonShaderMatcap = useTexture('/matcap-toon.png')
const whiteMatcap = useTexture('/matcap_white.png') 

useFrame((_, delta)=> {
  icoRef.rotation.x += 0.01
  icoRef.rotation.y += 0.01
  if (icoMaterialRef.shader) {
    icoMaterialRef.shader.uniforms.uTime.value += delta
  }
})
</script>

<T.PerspectiveCamera makeDefault position={[30, 0, 70]}>
  <OrbitControls />
</T.PerspectiveCamera>

{#await silverMatcap then map}
<T.Mesh position={[10, 0, 0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}

{#await rgbMatcap then map}
<T.Mesh position={[0,0,0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}

{#await pinkReflectionMatcap then map}
<T.Mesh position={[-10,0,0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}

{#await transparentReflectionMatcap then map}
<T.Mesh position={[-20,0,0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}

{#await toonShaderMatcap then map}
<T.Mesh position={[20,0,0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}

<T.AmbientLight intensity={0.1} />
<T.DirectionalLight position={[20, 0, 10]} intensity={0.5} />

<T.Mesh bind:ref={icoRef} position={[30,0,0]}>
  <T.BoxGeometry args={[3, 3, 3]} />
  <T.MeshPhongMaterial 
    bind:ref={icoMaterialRef}
    color='white'
    roughness={0.5}
    onBeforeCompile={(shader) => {
      shader.uniforms.uTime = { value: 0 }
      icoMaterialRef.shader = shader
      shader.fragmentShader = shader.fragmentShader.replace(
        `#include <color_pars_fragment>`,
        `#include <color_pars_fragment>
         uniform float uTime; 
`
      )
      shader.fragmentShader = shader.fragmentShader.replace(
        `#include <color_fragment>`,
        `#include <color_fragment>
         diffuseColor.rgb = vec3(1.0, 2.0, sin(uTime * 2.0));`
      )
    }}
  />
</T.Mesh>

{#await whiteMatcap then map}
<T.Mesh position={[-30,0,0]}>
  <T.SphereGeometry args={[3, 40, 40]} />
  <T.MeshMatcapMaterial 
    matcap={map}
  />
</T.Mesh>
{/await}
