<template>
  <div id="viewDiv" ref="viewDiv">
    <div v-if="popupIsOpen">
      <PopupDetail :cur="currPopupData"></PopupDetail>
    </div>

  </div>
</template>

<script setup>
import {ref, reactive, onMounted} from "vue";
import Map from "@arcgis/core/Map";
import esriConfig from "@arcgis/core/config.js";
import FeatureLayer from "@arcgis/core/layers/FeatureLayer"
esriConfig.assetsPath = "./assets";
import WebMap from "@arcgis/core/WebMap";
import SceneView from "@arcgis/core/views/SceneView";
import TileLayer from "@arcgis/core/layers/TileLayer";
import Basemap from "@arcgis/core/Basemap";
import MapView from "@arcgis/core/views/MapView";

import PopupDetail from "./PopupDetail.vue";


esriConfig.apiKey = "AAPK49e112a6eb8b4f35a9a75440b333e678-f61u8VzpSqYr32SlIW_fP5dH3e0giEnc56pe6OKeKcD1lntJ_Nf-1QKdkZ74iqg";
const viewDiv = ref(null);
let mapView = ref()
let drainageLayer;
let psUrl  = 'http://112.94.67.106:6080/arcgis/rest/services/app/PS_SPOUT_M/MapServer' // gzps
// feature  points 自定义样式
const trailheadsRenderer = {
  "type": "simple",
  "symbol": {
    "type": "picture-marker",
    "url": "http://static.arcgis.com/images/Symbols/NPS/npsPictograph_0231b.png",
    "width": "16px",
    "height": "16px"
  }
}
let highlightSelect;
let msg = ref('ssssssssssssssssssssss')
let currPopupData = reactive({});
const popupIsOpen = ref(false);

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
    baseLayers: [mapBaseLayerVec],
    // title: "矢量图",
    // id: "cva_w",
  });``
  const map = new Map({
    // basemap: vecBasemap,
    basemap: "osm",
  });
  mapView = new MapView({
    container: viewDiv.value,
    map: map,
    center: [-118.80543,34.02700],
    zoom: 13
  });
  mapView.ui.remove('attribution')

  // get points layer
  drainageLayer = new FeatureLayer({
    id: 'drainageLayer',
    // url: psUrl + '/1',
    url : 'https://services3.arcgis.com/GVgbJbqm8hXASVYi/arcgis/rest/services/Trailheads/FeatureServer/0',
    renderer: trailheadsRenderer,
    outFields: ['*']
  })
  mapView.map.add(drainageLayer)
  mapView.on('click', event => {
    mapView.hitTest(event).then(response => {
      let result = response.results[0];
      if(result?.graphic === undefined) {
        highlightSelect.remove()
        return
      }
      currPopupData = result.graphic.attributes;
      currPopupData.x = result.mapPoint.latitude;
      currPopupData.y = result.mapPoint.longitude;
      highLightFeature(result.graphic);
      gotoPosition(result.graphic)
      // queryFeatures(event)
      if( result.mapPoint.type === "point") detailPopup(currPopupData)
    })
  })
});

/*async function queryFeatures(screenPoints){
  console.log(screenPoints);
  const point = mapView.toMap(screenPoints);
  console.log(point);
  const query = drainageLayer.createQuery()
  query.geometry = point;
  // query.geometry = {
  //   type: 'point', // autocasts as new Point()
  //   longitude: screenPoints.mapPoint.longitude,
  //   latitude: screenPoints.mapPoint.latitude
  // }
  // query.spatialRelationship = 'within';
  query.spatialRelationship = 'intersects';
  query.distance = 6;
  query.returnQueryGeometry = true;
  query.outFields = ["*"]
  // query.returnGeometry = true;
  await drainageLayer.queryFeatures(query).then(featureSet => {
    console.log('ssssss', featureSet);
  })
}*/

// click feature highlight
function highLightFeature(graphic){
  mapView.whenLayerView(drainageLayer).then(layerView => {
    if(highlightSelect) highlightSelect.remove();
    highlightSelect = layerView.highlight(graphic)
  })
}

// click feature to the map center
function gotoPosition(feature){
  mapView.goTo({
            target: feature.geometry,
            tilt: 70,
            zoom: 13
          },
          {
            duration: 500,
            easing: "in-out-expo"
          })
      .then(()=>{
        popupIsOpen.value = true;
      })
      .catch((error) => {
        if (error.name != "AbortError") {
          console.error(error);
        }
      });
}

// feature detail popup
function detailPopup(data){
  // console.log(data);
  // const point = new Point({
  //   type: 'point',
  //   x: data.x,
  //   y: data.y,
  // })

  // console.log(popupIsOpen.value);
}

</script>



<style scoped>
#viewDiv {
  padding: 0;
  margin: 0;
  height: 100%;
  width: 100%;
}
</style>
