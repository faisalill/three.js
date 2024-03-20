<script>
  import { T, useFrame } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';

  let shaderMaterialRef;

  useFrame((_, delta)=>{
    shaderMaterialRef.uniforms.uTime.value += delta;
  })
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 6]}>
  <OrbitControls />
</T.PerspectiveCamera>

<T.Mesh>
  <T.PlaneGeometry args={[5,5]} />
  <T.ShaderMaterial 
    bind:ref={shaderMaterialRef}
    uniforms={{
      uTime: { value: 0.0 }
    }}
    vertexShader={`
      varying vec2 vUv;
      void main(){
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `}

    fragmentShader={`
      uniform float uTime;

      mat2 rot2D(float a) {
          return mat2(cos(a), -sin(a), sin(a), cos(a));
      }
      vec3 rotate3d (vec3 p, vec3 axis, float angle) {
        return mix(dot(axis,p) * axis, p, cos(angle)) + cross(axis, p) * sin(angle);
      }
      float opSmoothUnion( float d1, float d2, float k )
      {
          float h = clamp( 0.5 + 0.5*(d2-d1)/k, 0.0, 1.0 );
          return mix( d2, d1, h ) - k*h*(1.0-h);
      }
      float sdBox( vec3 p, vec3 b )
      {
        vec3 q = abs(p) - b;
        return length(max(q,0.0)) + min(max(q.x,max(q.y,q.z)),0.0);
      }
      float sdSphere( vec3 p, float s )
      {
        return length(p)-s;
      }
      vec3 palette(float t) {
        vec3 a = vec3(0.731, 1.098, 0.192);      
        vec3 b = vec3(0.358, 1.090, 0.657);
        vec3 c = vec3(1.077, 0.360, 0.328);
        vec3 d = vec3(0.965, 2.265, 0.837);

        return a + b*cos( 6.28318*(c*t+d) );
      }

      float getDistance(vec3 p){
        p.z += uTime * 0.5;
        p.y += 80.0;
        
        vec3 repeat = p;
        repeat.xy = fract(repeat.xy) - 0.5;
        repeat.z = mod(repeat.z, 0.40) - 0.20;

        vec3 cube = vec3(0.1);
        float dCube = sdBox(repeat , cube) ;
        
        float totalDistance = dCube;

        float ground = p.y + 0.75;
        return opSmoothUnion(ground, totalDistance, 0.5); 
      }

      varying vec2 vUv;
      void main(){
        vec2 newUv = vUv - 0.5;
        newUv *= 4.0;
        vec3 color = vec3(0);
      
        vec3 ray_origin = vec3(0.0, 0.0, -3.0);
        vec3 ray_direction = normalize(vec3(newUv, 1.0)); // 1 in z direction

        float distanceTravelled = 0.0;
        int i;
        for (i = 0; i < 80; i++){
          vec3 newPoint = ray_origin + ray_direction * distanceTravelled;
          
          newPoint.xy *= rot2D(distanceTravelled * 0.2 ) / 3.;
          newPoint.y += sin(distanceTravelled * 1.0) / 8.0;

          float distanceFromBox = getDistance(newPoint);
          if (distanceFromBox < 0.01){
            break;
          }
          distanceTravelled += distanceFromBox;
          if (distanceTravelled > 100.0){
            break;
          }
        }
        color = palette(distanceTravelled * 0.015 + float(i) * 0.009);
      
        
        gl_FragColor = vec4(color, 1.0);
      }
    `}
  />
</T.Mesh>
