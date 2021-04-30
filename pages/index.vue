<template>
  <div class="container"></div>
</template>

<script lang="ts">
import Vue from 'vue';
import {
  AmbientLight,
  AxesHelper,
  BoxGeometry,
  Clock,
  Color,
  GridHelper,
  Mesh,
  MeshToonMaterial,
  PerspectiveCamera,
  PointLight,
  PointLightHelper,
  Raycaster,
  Scene,
  Vector3,
  WebGLRenderer,
} from 'three';
import { PointerLockControls } from 'three/examples/jsm/controls/PointerLockControls.js';

export default Vue.extend({
  data() {
    return {
      scene: (null as unknown) as Scene,
      controls: (null as unknown) as PointerLockControls,
      camera: (null as unknown) as PerspectiveCamera,
      renderer: (null as unknown) as WebGLRenderer,
      clock: (null as unknown) as Clock,
      moveForward: false,
      moveBackward: false,
      moveLeft: false,
      moveRight: false,
      canJump: false,
      prevTime: 0,
      velocity: new Vector3(),
      direction: new Vector3(),
      raycaster: (null as unknown) as Raycaster,
    };
  },
  mounted() {
    this.init();
    this.animate();
  },
  methods: {
    init() {
      this.scene = new Scene();
      this.camera = new PerspectiveCamera(
        60,
        window.innerWidth / window.innerHeight,
        1,
        1000
      );
      this.renderer = new WebGLRenderer();

      this.renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(this.renderer.domElement);

      const geometry = new BoxGeometry(2, 2, 2);
      const material = new MeshToonMaterial({
        color: '#222',
      });
      const cube = new Mesh(geometry, material);
      this.scene.add(cube);
      this.scene.background = new Color('white');
      this.camera.position.z = 5;
      this.camera.position.y = 0.5;

      this.addControls();
      this.addHelpers();
      this.addLight();
    },
    addControls() {
      this.prevTime = performance.now();
      this.controls = new PointerLockControls(this.camera, document.body);

      window.addEventListener('click', () => {
        this.controls.lock();
      });

      this.scene.add(this.controls.getObject());

      const onKeyDown = (event: { code: any }) => {
        switch (event.code) {
          case 'ArrowUp':
          case 'KeyW':
            this.moveForward = true;
            break;

          case 'ArrowLeft':
          case 'KeyA':
            this.moveLeft = true;
            break;

          case 'ArrowDown':
          case 'KeyS':
            this.moveBackward = true;
            break;

          case 'ArrowRight':
          case 'KeyD':
            this.moveRight = true;
            break;

          case 'Space':
            this.canJump = true;
            break;
        }
      };

      const onKeyUp = (event: { code: any }) => {
        switch (event.code) {
          case 'ArrowUp':
          case 'KeyW':
            this.moveForward = false;
            break;

          case 'ArrowLeft':
          case 'KeyA':
            this.moveLeft = false;
            break;

          case 'ArrowDown':
          case 'KeyS':
            this.moveBackward = false;
            break;

          case 'ArrowRight':
          case 'KeyD':
            this.moveRight = false;
            break;
        }
      };

      document.addEventListener('keydown', onKeyDown);
      document.addEventListener('keyup', onKeyUp);

      // this.raycaster = new Raycaster(
      //   new Vector3(),
      //   new Vector3(0, -1, 0),
      //   0,
      //   10
      // );
    },
    addLight() {
      const light = new AmbientLight(0x888888);
      this.scene.add(light);
      const pointLight = new PointLight(0xffffff, 2, 800);
      this.scene.add(pointLight);
      pointLight.position.y = 10;
      const lightHelper = new PointLightHelper(pointLight, 1, 'black');
      this.scene.add(lightHelper);
    },
    addHelpers() {
      const size = 10;
      const divisions = 10;

      const gridHelper = new GridHelper(size, divisions);
      this.scene.add(gridHelper);
      const axesHelper = new AxesHelper(3);
      this.scene.add(axesHelper);
    },
    animate() {
      requestAnimationFrame(this.animate);

      const time = performance.now();

      if (this.controls.isLocked === true) {
        // this.raycaster.ray.origin.copy(this.controls.getObject().position);
        // this.raycaster.ray.origin.y -= 10;

        // const intersections = this.raycaster.intersectObjects(objects);

        const onObject = true;

        const delta = (time - this.prevTime) / 1000;

        this.velocity.x -= this.velocity.x * 10.0 * delta;
        this.velocity.z -= this.velocity.z * 10.0 * delta;

        this.velocity.y -= 9.8 * 100.0 * delta; // 100.0 = mass

        this.direction.z = Number(this.moveForward) - Number(this.moveBackward);
        this.direction.x = Number(this.moveRight) - Number(this.moveLeft);
        this.direction.normalize(); // this ensures consistent movements in all directions

        if (this.moveForward || this.moveBackward)
          this.velocity.z -= this.direction.z * 10.0 * delta;
        if (this.moveLeft || this.moveRight)
          this.velocity.x -= this.direction.x * 10.0 * delta;

        if (onObject === true) {
          this.velocity.y = Math.max(0, this.velocity.y);
          // canJump = true;
        }

        this.controls.moveRight(-this.velocity.x * delta);
        this.controls.moveForward(-this.velocity.z * delta);

        this.controls.getObject().position.y = 0.5;

        // this.controls.getObject().position.y += this.velocity.y * delta; // new behavior

        // if (this.controls.getObject().position.y < 10) {
        //   this.velocity.y = 0;
        //   this.controls.getObject().position.y = 10;

        //   // canJump = true;
        // }
      }

      this.prevTime = time;

      this.renderer.render(this.scene, this.camera);
    },
  },
});
</script>
