<script setup>
import {ref, onMounted} from "vue";
import Map from "@arcgis/core/Map";
import esriConfig from "@arcgis/core/config.js";
import FeatureLayer from "@arcgis/core/layers/FeatureLayer"

esriConfig.assetsPath = "./assets";
// import WebMap from "@arcgis/core/WebMap";
import SceneView from "@arcgis/core/views/SceneView";
import TileLayer from "@arcgis/core/layers/TileLayer";
import Basemap from "@arcgis/core/Basemap";
import MapView from "@arcgis/core/views/MapView";

esriConfig.apiKey =
    "AAPK49e112a6eb8b4f35a9a75440b333e678-f61u8VzpSqYr32SlIW_fP5dH3e0giEnc56pe6OKeKcD1lntJ_Nf-1QKdkZ74iqg";

const viewDiv = ref(null);
let mapView = ref()
let drainageLayer;
let psUrl  = 'http://112.94.67.106:6080/arcgis/rest/services/app/PS_SPOUT_M/MapServer'
onMounted(() => {

  /*
    // 三维图层 DEMO
    const map = new Map({
      basemap: "arcgis-topographic", // Basemap layer service
      ground: "world-elevation", //Elevation service
    });

    const sceneMap = new SceneView({
      container: viewDiv.value,
      map: map,
      camera: {
        position: {
          x: -118.808, //Longitude
          y: 33.961, //Latitude
          z: 2000, //Meters
        },
        tilt: 75,
      },
    });
    sceneMap.when(() => {
      console.log("地图加载成功");
    });*/

  const mapBaseLayerVec = new TileLayer({
    url: "http://112.94.67.106:6080/proxy/80cec1371ca6f576/4dfa4360488bfc42",
  });
  const vecBasemap = new Basemap({
    // baseLayers: [mapBaseLayerVec],
    basemap: "arcgis-topographic",

    title: "矢量图",
    id: "cva_w",
  });
  const map = new Map({
    basemap: vecBasemap,
  });
  mapView = new MapView({
    container: viewDiv.value,
    map: map,
    center: [-118.80543,34.02700],
    zoom: 13
  });
  mapView.ui.remove('attribution')
  mapView.on('click', e => {
    mapView.hitTest(e).then(response => {
      console.log(response);
    })
  })
  drainageLayer = new FeatureLayer({
    // id: 'drainageLayer',
    // url: psUrl + '/1',
    // outFields: ['*']
    url : 'https://services3.arcgis.com/GVgbJbqm8hXASVYi/arcgis/rest/services/Trailheads/FeatureServer/0'
  })
  console.log(drainageLayer);
  mapView.map.add(drainageLayer)
});

</script>

<template>
  <div id="viewDiv" ref="viewDiv"></div>
</template>

<style scoped>
#viewDiv {
  padding: 0;
  margin: 0;
  height: 100%;
  width: 100%;
}
</style>
