<script>
  import * as THREE from "three";
  import { onMount } from "svelte";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
  import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";

  let scene, camera, renderer;
  let model;

  onMount(() => {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );

    camera.position.x = 1;
    camera.position.y = 1;
    camera.position.z = 24;

    renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.outputEncoding = THREE.sRGBEncoding;
    renderer.toneMapping = THREE.CineonToneMapping;
    renderer.toneMappingExposure = 1.5;
    document.body.appendChild(renderer.domElement);

    // const gridHelper = new THREE.GridHelper(10, 10);
    // scene.add(gridHelper);
    //
    // const axesHelper = new THREE.AxesHelper(50);
    // scene.add(axesHelper);
    //
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.update();

    const geometry = new THREE.PlaneGeometry(10, 10, 10, 10);
    const material = new THREE.PointsMaterial({ 
      // color: 0x888888,
      size: 0.22,
      sizeAttenuation: true
    });

    const points = new THREE.Points(geometry, material);
    points.position.set(-14, 0, 0);
    scene.add(points);

    const geometry2 = new THREE.BufferGeometry();
    const vertices = [];
    const colors = [];
    const count = 1200;
    for (let i = 0; i < count; i++) {
      const x = (Math.random() - 0.5) * 10;
      const y = (Math.random() - 0.5) * 10;
      const z = (Math.random() - 0.5) * 10;
      vertices.push(x, y, z);
      colors.push((Math.random() - 0.5), (Math.random() - 0.5), (Math.random() - 0.5));
    }

    geometry2.setAttribute(
      "position",
      new THREE.Float32BufferAttribute(vertices, 3)
    );

    geometry2.setAttribute(
      "color",
      new THREE.Float32BufferAttribute(colors, 3)
    );

    const particleTexture = new THREE.TextureLoader().load(
      "/circle.png"
    );

    const material2 = new THREE.PointsMaterial({
      vertexColors: true,
      size: 0.1,
      sizeAttenuation: true,
      transparent: true,
      alphaMap: particleTexture,
      // alphaTest: 0.001,
      // depthTest: false,
      depthWrite: false,
      blending: THREE.AdditiveBlending,
    });

    const points2 = new THREE.Points(geometry2, material2);
    points2.position.set(0, 0, 0);
    scene.add(points2);

    const geometry3 = new THREE.BoxGeometry(4, 4, 4);
    const material3 = new THREE.PointsMaterial({
      color: 0x888888,
      size: 0.12,
      sizeAttenuation: true,
      transparent: true,
    });
    const cube = new THREE.Points(geometry3, material3);
    cube.position.set(0, 0, 2);
    scene.add(cube);

    const clock = new THREE.Clock();
    function animate() {
      requestAnimationFrame(animate);

      points.rotation.z += 0.001;
      material.size = 0.05 + Math.sin(Date.now() * 0.001) * 0.05;

      for (let i = 0; i < count; i++) {
        const i3 = i * 3;
        const x = geometry2.attributes.position.array[i3];
        const y = geometry2.attributes.position.array[i3 + 1];
        const z = geometry2.attributes.position.array[i3 + 2];

        geometry2.attributes.position.array[i3 + 1] =
        Math.sin(clock.getElapsedTime() + x)
        // geometry2.attributes.position.array[i3] = x + Math.sin(Date.now() * 0.01 + i) * 0.01;
        // geometry2.attributes.position.array[i3 + 1] = y + Math.sin(Date.now() * 0.01 + i) * 0.01;
        // geometry2.attributes.position.array[i3 + 2] = z + Math.sin(Date.now() * 0.01 + i) * 0.01;
      }
      geometry2.attributes.position.needsUpdate = true;

      controls.update();

      renderer.render(scene, camera);
    }
    animate();
  });
</script>
