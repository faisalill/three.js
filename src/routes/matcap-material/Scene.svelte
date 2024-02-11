<script>
import { T, useFrame } from '@threlte/core';
import { OrbitControls, useTexture } from '@threlte/extras';

const silverMatcap = useTexture('/matcap-silver.jpg')
const rgbMatcap = useTexture('matcap-rgb.png') 
const pinkReflectionMatcap = useTexture('matcap-pink-reflection.png')
const transparentReflectionMatcap = useTexture('matcap-transparent.png')
const toonShaderMatcap = useTexture('matcap-toon.png')

</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 50]} >
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
