<script>
  import { T, useFrame } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';

  let shaderMaterialRef;

  useFrame((_, delta)=>{
    shaderMaterialRef.uniforms.u_time.value += delta;
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
      u_time: { value: 0.0 }
    }}
    vertexShader={`
      varying vec2 vUv;
      void main(){
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `}

    fragmentShader={`
      mat2 rot2D(float a) {
          return mat2(cos(a), -sin(a), sin(a), cos(a));
      }
      vec3 rotate3d (vec3 p, vec3 axis, float angle) {
        return mix(dot(axis,p) * axis, p, cos(angle)) + cross(axis, p) * sin(angle);
      }

      uniform float u_time;

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

      float getDistance(vec3 p){

        p.z -= sin(u_time);
        
        vec3 repeat = p;
        repeat = fract(repeat) - 0.5;

        vec3 cube = vec3(0.1);
        float dCube = sdBox(repeat , cube) ;
        
        float sphere = 1.0;
        vec3 spherePosition = vec3( sin(u_time) * 3.0, 0.0, 0.0);
        float dSphere = sdSphere(p - spherePosition, sphere);

        float smoothFactor = 1.0;
        float totalDistance = opSmoothUnion(dCube, dSphere, smoothFactor);

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

          float distanceFromBox = getDistance(newPoint);
          if (distanceFromBox < 0.01){
            break;
          }
          distanceTravelled += distanceFromBox;
          if (distanceTravelled > 100.0){
            break;
          }
        }
        color = vec3(distanceTravelled * 0.1 + float(i) * 0.01);
    
        gl_FragColor = vec4(color, 1.0);
      }
    `}
  />
</T.Mesh>
