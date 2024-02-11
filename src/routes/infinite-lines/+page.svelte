<script>
  import { Canvas } from '@threlte/core';
  import Scene from './Scene.svelte';
</script>

<div class="canvas-wrapper">
<Canvas>
  <Scene />
</Canvas>
</div>

<style>
  .canvas-wrapper {
    width: 100vw;
    height: 100vh;
  }
</style>
