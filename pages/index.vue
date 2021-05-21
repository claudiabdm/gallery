<template>
  <div class="container"></div>
</template>

<script lang="ts">
import Vue from 'vue';
import {
  AmbientLight,
  AxesHelper,
  BoxGeometry,
  Color,
  DoubleSide,
  GridHelper,
  Group,
  Mesh,
  MeshStandardMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  Raycaster,
  RepeatWrapping,
  Scene,
  SpotLight,
  SpotLightHelper,
  Texture,
  TextureLoader,
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
      moveForward: false,
      moveBackward: false,
      moveLeft: false,
      moveRight: false,
      canJump: false,
      prevTime: 0,
      velocity: new Vector3(),
      direction: new Vector3(),
      raycaster: (null as unknown) as Raycaster,
      objects: [] as any[],
      image: 'image.jpg',
      plane: (null as unknown) as PlaneGeometry,
      loader: (null as unknown) as TextureLoader,
      wallMaterial: (null as unknown) as MeshStandardMaterial,
    };
  },
  mounted() {
    this.init();
    this.animate();
  },
  methods: {
    init() {
      this.loader = new TextureLoader();
      this.scene = new Scene();
      this.camera = new PerspectiveCamera(
        60,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      this.renderer = new WebGLRenderer({
        antialias: true,
      });

      this.renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(this.renderer.domElement);
      this.scene.background = new Color('#eee');

      this.camera.position.z = 5;
      this.camera.position.y = 1;
      this.camera.position.x = 0;

      this.raycaster = new Raycaster(
        new Vector3(),
        new Vector3(0, -1, 0),
        0,
        10
      );

      this.plane = new PlaneGeometry(1, 1);
      this.wallMaterial = this.createWallMaterial({ width: 10, height: 2 });

      this.addControls();
      this.addHelpers();
      this.addLight();
      this.addFloor();
      this.addCeiling();
      this.addRoomWalls();
      this.addExpoWalls();
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
    },
    addFloor() {
      const geometry = this.plane;
      const map = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_Color.jpg')
      );
      const normalMap = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_Normal.jpg')
      );
      const displacementMap = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_Displacement.jpg')
      );
      const roughnessMap = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_Roughness.jpg')
      );
      const metalnessMap = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_Metalness.jpg')
      );
      const aoMap = this.loader.load(
        require('~/assets/WoodFloor044_1K-JPG/WoodFloor044_1K_AmbientOcclusion.jpg')
      );

      const maps: { [key: string]: Texture } = {
        normalMap,
        map,
        displacementMap,
        roughnessMap,
        metalnessMap,
        aoMap,
      };

      for (const mapType in maps) {
        maps[mapType].wrapS = maps[mapType].wrapT = RepeatWrapping;
        maps[mapType].repeat.set(10, 10);
      }

      const material = new MeshStandardMaterial({
        // color: '#fff',
        side: DoubleSide,
        ...maps,
      });
      const floor = new Mesh(geometry, material);
      floor.rotateX(-Math.PI / 2);
      floor.scale.set(10, 10, 0.01);
      this.scene.add(floor);
    },
    addCeiling() {
      const geometry = this.plane;
      const map = this.loader.load(
        require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Color.jpg')
      );
      const normalMap = this.loader.load(
        require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Normal.jpg')
      );
      const displacementMap = this.loader.load(
        require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Displacement.jpg')
      );
      const roughnessMap = this.loader.load(
        require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Roughness.jpg')
      );
      // const metalnessMap = this.loader.load(
      //   require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Metalness.jpg')
      // );
      // const aoMap = this.loader.load(
      //   require('~/assets/Plaster001_2k-JPG/Plaster001_2k_AmbientOcclusion.jpg')
      // );
      // const emissiveMap = this.loader.load(
      //   require('~/assets/Plaster001_2k-JPG/Plaster001_2k_Emission.jpg')
      // );

      const maps: { [key: string]: Texture } = {
        normalMap,
        map,
        displacementMap,
        roughnessMap,
        // metalnessMap,
        // aoMap,
        // emissiveMap,
      };

      for (const mapType in maps) {
        maps[mapType].wrapS = maps[mapType].wrapT = RepeatWrapping;
        maps[mapType].repeat.set(10, 10);
      }

      const material = new MeshStandardMaterial({
        // color: '#fff',
        side: DoubleSide,
        ...maps,
        roughness: 1,
        // metalness: 0.5,
        emissive: new Color(0x666666),
      });

      // material.displacementBias = -0.36;
      material.displacementBias = -0.09;
      const ceil = new Mesh(geometry, material);
      ceil.rotateX(-Math.PI / 2);
      ceil.scale.set(10, 10, 4.0);
      this.scene.add(ceil);
    },
    addLight() {
      const light = new AmbientLight(0x888888, 1.0);
      this.scene.add(light);

      // RectAreaLightUniformsLib.init();
      // for (let i = 1; i <= 6; i++) {
      //   const rectAreaLight = new RectAreaLight(0xffffff, 10.0, 0.5, 1.0);
      //   const positionX = i > 3 ? (i - 3) * 2.5 - 5 : i * 2.5 - 5;
      //   const positionZ = i > 3 ? -2.5 : 2.5;
      //   rectAreaLight.position.set(positionX, 1.9, positionZ);
      //   rectAreaLight.rotateX(-Math.PI / 2);
      //   const rectAreaLightHelper = new RectAreaLightHelper(rectAreaLight);
      //   rectAreaLight.add(rectAreaLightHelper);
      //   this.scene.add(rectAreaLight);
      // }
      // const lightHelper = new SpotLightHelper(pointLight, 'blue');
      // this.scene.add(lightHelper);
    },
    addHelpers() {
      const size = 10;
      const divisions = 10;

      const gridHelper = new GridHelper(size, divisions);
      this.scene.add(gridHelper);
      const axesHelper = new AxesHelper(3);
      this.scene.add(axesHelper);
    },
    addExpoWalls() {
      const geometry = new PlaneGeometry(5, 2);
      const material = this.wallMaterial;
      const wall = new Mesh(geometry, material);
      wall.position.y = 1.0;

      const imageGeometry = new BoxGeometry(0.5, 0.5, 0.05);
      const imageTexture = this.loader.load(this.image);
      const imgMaterial = new MeshStandardMaterial({
        map: imageTexture,
      });

      // const frameGeometry = new BoxGeometry(0.5, 0.5, 0.0);
      // // const frameTexture = this.loader.load(this.frame);
      // const frameMaterial = new MeshBasicMaterial({
      //   // map: frameTexture,
      // });

      const image = new Mesh(imageGeometry, imgMaterial);
      image.position.z = 0.01;

      const pointLight = new SpotLight(0xffffff, 1.5, 3.0, 0.25, 0.35, 1.0);
      pointLight.position.set(0.0, 2.25, 1.5);
      // pointLight.rotateZ(Math);
      pointLight.target = image;
      this.scene.add(pointLight);
      this.scene.add(pointLight.target);
      const lightHelper = new SpotLightHelper(pointLight, 'blue');
      this.scene.add(lightHelper);
      wall.add(image);
      this.scene.add(wall);
      this.objects.push(wall);
    },
    addRoomWalls() {
      const roomWalls = new Group();
      const geometry = new PlaneGeometry(10, 2);
      const material = this.wallMaterial;
      const wall1 = new Mesh(geometry, material);
      wall1.position.z = -5;
      roomWalls.add(wall1);

      const wall2 = new Mesh(geometry, material);
      wall2.position.z = 5;
      roomWalls.add(wall2);

      const wall3 = new Mesh(geometry, material);
      wall3.rotateY(Math.PI / 2);
      wall3.position.x = -5;
      roomWalls.add(wall3);

      const wall4 = new Mesh(geometry, material);
      wall4.rotateY(Math.PI / 2);
      wall4.position.x = 5;
      roomWalls.add(wall4);

      roomWalls.position.y = 1.0;

      this.scene.add(roomWalls);
    },
    createWallMaterial({ width = 0, height = 0 }): MeshStandardMaterial {
      const map = this.loader.load(
        require('~/assets/Bricks060_2K-JPG/Bricks060_2K_Color.jpg')
      );
      const normalMap = this.loader.load(
        require('~/assets/Bricks060_2K-JPG/Bricks060_2K_Normal.jpg')
      );
      const displacementMap = this.loader.load(
        require('~/assets/Bricks060_2K-JPG/Bricks060_2K_Displacement.jpg')
      );
      const roughnessMap = this.loader.load(
        require('~/assets/Bricks060_2K-JPG/Bricks060_2K_Roughness.jpg')
      );
      const aoMap = this.loader.load(
        require('~/assets/Bricks060_2K-JPG/Bricks060_2K_AmbientOcclusion.jpg')
      );
      const maps: { [key: string]: Texture } = {
        normalMap,
        map,
        displacementMap,
        roughnessMap,
        aoMap,
      };

      for (const mapType in maps) {
        maps[mapType].wrapS = maps[mapType].wrapT = RepeatWrapping;
        maps[mapType].repeat.set(width, height);
      }

      const material = new MeshStandardMaterial({
        // color: '#fff',
        side: DoubleSide,
        ...maps,
        roughnessMap,
        // roughness: 1.0,
        // emissive: new Color(0x666666),s
      });

      material.displacementScale = 0.0;

      return material;
    },
    animate() {
      requestAnimationFrame(this.animate);

      const time = performance.now();

      if (this.controls.isLocked === true) {
        this.raycaster.ray.origin.copy(this.controls.getObject().position);
        this.raycaster.ray.origin.y -= 10;

        const intersections = this.raycaster.intersectObjects(this.objects);

        const onObject = intersections.length > 0;

        const delta = (time - this.prevTime) / 1000;

        this.velocity.x -= this.velocity.x * 100.0 * delta;
        this.velocity.z -= this.velocity.z * 100.0 * delta;

        // this.velocity.y -= 9.8 * 1.0 * delta; // 100.0 = mass

        this.direction.z = Number(this.moveForward) - Number(this.moveBackward);
        this.direction.x = Number(this.moveRight) - Number(this.moveLeft);
        this.direction.normalize(); // this ensures consistent movements in all directions

        if (this.moveForward || this.moveBackward)
          this.velocity.z -= this.direction.z * 100.0 * delta;
        if (this.moveLeft || this.moveRight)
          this.velocity.x -= this.direction.x * 100.0 * delta;

        if (onObject === true) {
          // this.velocity.y = Math.max(0, this.velocity.y);
          // canJump = true;
        }

        this.controls.moveRight(-this.velocity.x * delta);
        this.controls.moveForward(-this.velocity.z * delta);

        this.controls.getObject().position.y = 1.0;

        this.controls.getObject().position.y += this.velocity.y * delta; // new behavior

        // if (this.controls.getObject().position.y < 10) {
        //   this.velocity.y = 0;
        //   this.controls.getObject().position.y = 0.5;

        //   // canJump = true;
        // }
      }

      this.prevTime = time;

      this.renderer.render(this.scene, this.camera);
    },
  },
});
</script>
