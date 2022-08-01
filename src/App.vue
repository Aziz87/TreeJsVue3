<template>
  <div :key="json" style="height: 100vh">
    <Renderer ref="renderer" :orbit-ctrl="{ enableDamping: true }" :pixelRatio="1" antialias autoClear pointer resize
              shadow xr>
      <Camera ref="camera" :position="{ z: 20, x:0, y: 5}">
        <Scene ref="scene" background="#fff">
          <AmbientLight :intensity=".4"/>
          <DirectionalLight :intensity=".8 * Math.PI" :position="{x:0.5, y:0, z:0.866}"/>
          <Group v-for="(x,i) of [2]">
            <Group v-for="(personage,j) of ['COSSACK','JANISSARY']">
              <GltfModel v-for="file of getFiles(personage, x,false)" :key="file" :position="{y:-5, x:i*10, z:-j*10}"
                         :src="file" @beforeLoad="initDraco"/>
            </Group>
          </Group>
        </Scene>
      </Camera>
    </Renderer>
  </div>
</template>
<script>
import {DRACOLoader} from 'three/examples/jsm/loaders/DRACOLoader';
import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader';
import {MeshoptDecoder} from 'three/examples/jsm/libs/meshopt_decoder.module.js';
import {PMREMGenerator, sRGBEncoding} from 'three';
import axios from "axios";
import {
  AmbientLight,
  Box,
  BoxGeometry,
  Camera,
  DirectionalLight,
  GltfModel,
  HemisphereLight,
  LambertMaterial,
  MatcapMaterial,
  Plane,
  PointLight,
  RectAreaLight,
  Renderer,
  Scene,
  SpotLight
} from 'troisjs';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/js/libs/draco/'); // use a full url path
export default {
  components: {
    AmbientLight, DirectionalLight,
    HemisphereLight, MatcapMaterial,
    RectAreaLight,
    Camera, BoxGeometry,
    GltfModel,
    Renderer, Box,
    Scene,
    SpotLight,
    PointLight,
    Plane,
    LambertMaterial
  },

  data() {
    let json = [];
    return {json}
  },

  mounted() {

    (async () => {
      await this.parseJsonb();
      await this.renderSetting();
      await this.updateLights();
    })();

  },

  methods: {

    updateLights() {
      const renderer = this.$refs.renderer.renderer;
      renderer.physicallyCorrectLights = true;
      renderer.outputEncoding = sRGBEncoding;
    },

    renderSetting() {
      const renderer = this.$refs.renderer.renderer;
      const scene = this.$refs.scene.scene;
      const pmremGenerator = new PMREMGenerator(renderer);
      pmremGenerator.compileEquirectangularShader();

      new RGBELoader().load('static/hdr/2k.hdr', (texture) => {
        const envMap = pmremGenerator.fromEquirectangular(texture).texture;
        // const envMap = pmremGenerator.fromScene(scene).texture;
        pmremGenerator.dispose();
        scene.environment = envMap;
      })
    },

    async parseJsonb() {
      const json = await axios.get("/static/files.json");
      this.json = json.data;
    },

    //"COSSACK", 4, true
    getFiles(personage, setting, starterDress = false) {
      const files = [];
      if (this.json[personage]) for (let category of Object.keys(this.json[personage])) {
        if (starterDress && category.indexOf("DRESS") === 0) continue;
        else if (!starterDress && category.indexOf("STARTER") === 0) continue;
        let file = this.json[personage][category][setting - 1]
        files.push(`/models/Draco1024Webp/${personage}/${category}/${file}?a`)
      }
      return files;
    },
    initDraco(loader) {
      loader
          .setCrossOrigin('anonymous')
          .setMeshoptDecoder(MeshoptDecoder)
          .setDRACOLoader(dracoLoader);
    },
    onError(err) {
      console.log(err)
    }
  }
};
</script>
<style>
body {
  margin: 0
}
</style>
