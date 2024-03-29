<script>
  import * as THREE from 'three';
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
  import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
  import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
   import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
  import {onMount} from 'svelte';

  let camera, scene, renderer, renderPass, composer, bloomPass;
  let geometry, material, mesh;
  let audioContext, sound;

  onMount(()=>{

    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

    renderer = new THREE.WebGLRenderer();
    renderer.setSize( window.innerWidth, window.innerHeight );
    document.body.appendChild( renderer.domElement );

    renderPass = new RenderPass( scene, camera );
    composer = new EffectComposer( renderer );
    composer.addPass( renderPass );

    bloomPass = new UnrealBloomPass( new THREE.Vector2( window.innerWidth, window.innerHeight ), 0.8, 2, 0.1);
    composer.addPass( bloomPass );

    renderer.toneMapping = THREE.CineonToneMapping;
    renderer.toneMappingExposure = 1;

    const uniforms = {
      u_frequency: { type: 'f', value: 0.0},
      u_time: { type: 'f', value: 0.0},
      u_resolution: { type: 'v2', value: new THREE.Vector2(window.innerWidth, window.innerHeight) },
    }

    geometry = new THREE.IcosahedronGeometry( 4, 30);
    material = new THREE.ShaderMaterial( { 
      uniforms: uniforms,
      vertexShader: `
        varying vec2 vUv;
        varying vec3 vNormal;
        varying vec3 vPosition;
        varying float vFrequency;

        uniform float u_time;
    
        vec3 mod289(vec3 x)
        {
          return x - floor(x * (1.0 / 289.0)) * 289.0;
        }

        vec4 mod289(vec4 x)
        {
          return x - floor(x * (1.0 / 289.0)) * 289.0;
        }

        vec4 permute(vec4 x)
        {
          return mod289(((x*34.0)+10.0)*x);
        }

        vec4 taylorInvSqrt(vec4 r)
        {
          return 1.79284291400159 - 0.85373472095314 * r;
        }

        vec3 fade(vec3 t) {
          return t*t*t*(t*(t*6.0-15.0)+10.0);
        }   

         float pnoise(vec3 P, vec3 rep)
        {
          vec3 Pi0 = mod(floor(P), rep); // Integer part, modulo period
          vec3 Pi1 = mod(Pi0 + vec3(1.0), rep); // Integer part + 1, mod period
          Pi0 = mod289(Pi0);
          Pi1 = mod289(Pi1);
          vec3 Pf0 = fract(P); // Fractional part for interpolation
          vec3 Pf1 = Pf0 - vec3(1.0); // Fractional part - 1.0
          vec4 ix = vec4(Pi0.x, Pi1.x, Pi0.x, Pi1.x);
          vec4 iy = vec4(Pi0.yy, Pi1.yy);
          vec4 iz0 = Pi0.zzzz;
          vec4 iz1 = Pi1.zzzz;

          vec4 ixy = permute(permute(ix) + iy);
          vec4 ixy0 = permute(ixy + iz0);
          vec4 ixy1 = permute(ixy + iz1);

          vec4 gx0 = ixy0 * (1.0 / 7.0);
          vec4 gy0 = fract(floor(gx0) * (1.0 / 7.0)) - 0.5;
          gx0 = fract(gx0);
          vec4 gz0 = vec4(0.5) - abs(gx0) - abs(gy0);
          vec4 sz0 = step(gz0, vec4(0.0));
          gx0 -= sz0 * (step(0.0, gx0) - 0.5);
          gy0 -= sz0 * (step(0.0, gy0) - 0.5);

          vec4 gx1 = ixy1 * (1.0 / 7.0);
          vec4 gy1 = fract(floor(gx1) * (1.0 / 7.0)) - 0.5;
          gx1 = fract(gx1);
          vec4 gz1 = vec4(0.5) - abs(gx1) - abs(gy1);
          vec4 sz1 = step(gz1, vec4(0.0));
          gx1 -= sz1 * (step(0.0, gx1) - 0.5);
          gy1 -= sz1 * (step(0.0, gy1) - 0.5);

          vec3 g000 = vec3(gx0.x,gy0.x,gz0.x);
          vec3 g100 = vec3(gx0.y,gy0.y,gz0.y);
          vec3 g010 = vec3(gx0.z,gy0.z,gz0.z);
          vec3 g110 = vec3(gx0.w,gy0.w,gz0.w);
          vec3 g001 = vec3(gx1.x,gy1.x,gz1.x);
          vec3 g101 = vec3(gx1.y,gy1.y,gz1.y);
          vec3 g011 = vec3(gx1.z,gy1.z,gz1.z);
          vec3 g111 = vec3(gx1.w,gy1.w,gz1.w);

          vec4 norm0 = taylorInvSqrt(vec4(dot(g000, g000), dot(g010, g010), dot(g100, g100), dot(g110, g110)));
          g000 *= norm0.x;
          g010 *= norm0.y;
          g100 *= norm0.z;
          g110 *= norm0.w;
          vec4 norm1 = taylorInvSqrt(vec4(dot(g001, g001), dot(g011, g011), dot(g101, g101), dot(g111, g111)));
          g001 *= norm1.x;
          g011 *= norm1.y;
          g101 *= norm1.z;
          g111 *= norm1.w;

          float n000 = dot(g000, Pf0);
          float n100 = dot(g100, vec3(Pf1.x, Pf0.yz));
          float n010 = dot(g010, vec3(Pf0.x, Pf1.y, Pf0.z));
          float n110 = dot(g110, vec3(Pf1.xy, Pf0.z));
          float n001 = dot(g001, vec3(Pf0.xy, Pf1.z));
          float n101 = dot(g101, vec3(Pf1.x, Pf0.y, Pf1.z));
          float n011 = dot(g011, vec3(Pf0.x, Pf1.yz));
          float n111 = dot(g111, Pf1);

          vec3 fade_xyz = fade(Pf0);
          vec4 n_z = mix(vec4(n000, n100, n010, n110), vec4(n001, n101, n011, n111), fade_xyz.z);
          vec2 n_yz = mix(n_z.xy, n_z.zw, fade_xyz.y);
          float n_xyz = mix(n_yz.x, n_yz.y, fade_xyz.x); 
          return 2.2 * n_xyz;
        } 

        uniform float u_frequency;

        void main() {
          float noise = pnoise(position + u_time, vec3(20.0));
          float displacement = (u_frequency * 0.03) * noise * 0.4;
          vec3 newPosition = position + normal * displacement;

          gl_Position = projectionMatrix * modelViewMatrix * vec4(newPosition, 1.0 );
        }
      `,
      fragmentShader: `
        varying vec2 vUv;
        varying vec3 vNormal;
        varying vec3 vPosition;

        uniform float u_frequency;
        uniform float u_time;
        uniform vec2 u_resolution;


        void main() {
          vec2 normalize = gl_FragCoord.xy / u_resolution.xy;


          // Create a colorful pattern using sine and cosine functions
          float red = 0.5 + 0.5 * sin((u_frequency * 0.02) * normalize.x + u_time);
          float green = 0.5 + 0.5 * sin((u_frequency * 0.02) * normalize.y + u_time);
          float blue = 0.5 + 0.5 * sin((u_frequency * 0.02) * (normalize.x + normalize.y) + u_time);

          // Combine the color components
          vec3 color = vec3(red, green, blue);

          gl_FragColor = vec4(color, 1.0);
        }
      `,
      wireframe: true} );
    mesh = new THREE.Mesh( geometry, material );
    mesh.rotation.x = -Math.PI/2;
    scene.add( mesh );

    camera.position.z = 10;
    camera.position.x = 3;
    camera.position.y = 3;

    const controls = new OrbitControls( camera, renderer.domElement );
    controls.update();

    audioContext = new window.AudioContext(); 

    const listener = new THREE.AudioListener();
    camera.add( listener );

    sound = new THREE.Audio( listener );

    const audioLoader = new THREE.AudioLoader();
    audioLoader.load( '/big_fish.mp3', function( buffer ) {
      sound.setBuffer( buffer );
    });

    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        audioContext.resume();
        sound.play();
      }
    })

    const analyser = new THREE.AudioAnalyser( sound, 32 );

    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame( animate );

      uniforms.u_time.value = clock.getElapsedTime();
      uniforms.u_frequency.value = analyser.getAverageFrequency()

      controls.update();
      // renderer.render( scene, camera );
      composer.render();
    }

    animate();
  })

</script>

<div
class="fixed left-[39%] bottom-12"
>
<button
class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
on:click={() => {
  audioContext.resume();
  sound.play();
}}
>
  Play
</button>

<button
class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
on:click={() => {
  sound.pause();
}}
>
  Pause
</button>
</div>
