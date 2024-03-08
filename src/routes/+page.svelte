<script lang="ts">
  import { onMount } from 'svelte';
  import {
    Scene,
    PerspectiveCamera,
    WebGLRenderer,
    Object3D,
    AmbientLight,
    DirectionalLight,
    HemisphereLight,
    LinearToneMapping,
    PMREMGenerator,
    BoxGeometry,
    Mesh,
    MeshBasicMaterial,
  } from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
  import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment.js'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
  let controls: OrbitControls
  let renderedSceneElement: Element
  let scene: Scene
  let renderer: WebGLRenderer
  let camera: PerspectiveCamera

  onMount(async () => {
    console.log('mounted');

    scene = new Scene()
    renderer = new WebGLRenderer()

    renderedSceneElement = document.getElementById('scene-viewer')?.appendChild(renderer.domElement)

    camera = new PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

    renderer.setSize( window.innerWidth, window.innerHeight );
    renderer.toneMapping = LinearToneMapping
    renderer.toneMappingExposure = 1.0

    const envMap = new PMREMGenerator(renderer).fromScene(new RoomEnvironment()).texture
    scene.environment = envMap
    scene.background = envMap

    camera.position.z = 5;
    scene.add(camera)

    addLights(camera, scene)

    const meshScene = await loadGLTF(`models/Prefab_Foliage_Tree_RedMaple_Leaves.glb`)
    scene.add(meshScene.children[0])

    controls = new OrbitControls(camera, renderedSceneElement)

    const geometry = new BoxGeometry( 1, 1, 1 );
    const material = new MeshBasicMaterial( { color: 0x00ff00 } );
    const cube = new Mesh( geometry, material );
    scene.add( cube );

    animate();
  });

  const addLights = (camera: PerspectiveCamera, scene: Scene) => {
    const hemiLight = new HemisphereLight()
    const ambientLight = new AmbientLight(0xffffff, 0.3)
    const directionalLight = new DirectionalLight('#FFFFFF', 0.8 * Math.PI)
    directionalLight.name = 'MainLight'
    directionalLight.position.set(0.5, 0, 0.866); // ~60º
    scene.add(hemiLight)
    if (camera) {
      camera.add(ambientLight)
      camera.add(directionalLight)
    }
  }

  export const loadGLTF = async (pathToFile: string): Promise<Object3D> => {
    return new Promise<Object3D>(async (resolve, reject) => {
      const loader = new GLTFLoader(undefined)
      const gltf = await loader.loadAsync(pathToFile)
      let scene: Object3D = gltf.scene
      scene.position.x = 0
      scene.position.y = 0
      scene.position.z = 0
      scene.rotation.y = Math.PI / 2
      resolve(scene)
    })
  }


  export const animate = () => {
    requestAnimationFrame( animate );
    controls.update()
    renderer.render( scene, camera );
  }





</script>
<div style="height: 100%; width: 100%;" id="scene-viewer" />
