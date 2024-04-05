<template>
  <ol-map ref="mapRef" style="width: 100%; height: 100%;">
    <ol-view ref="view" :center="center" :zoom="zoom" :projection="projection" />

    <ol-tile-layer ref="osmLayer" title="OSM" :visible="false">
      <ol-source-osm />
    </ol-tile-layer>

    <ol-tile-layer ref="photoJapanLayer" title="地理院空中写真" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://cyberjapandata.gsi.go.jp/xyz/seamlessphoto/{z}/{x}/{y}.jpg"
        maxZoom="18" attributions="国土地理院" />
    </ol-tile-layer>

    <ol-tile-layer ref="cyberJapanLayer" title="地理院地図">
      <ol-source-xyz crossOrigin="anonymous" url="https://cyberjapandata.gsi.go.jp/xyz/std/{z}/{x}/{y}.png" maxZoom="18"
        attributions="国土地理院" />
    </ol-tile-layer>

    <ol-tile-layer ref="reliefLayer" title="色別標高図" :opacity="0.5">
      <ol-source-xyz crossOrigin="anonymous" url="https://maps.gsi.go.jp/xyz/relief/{z}/{x}/{y}.png"
        maxZoom="15" attributions="国土地理院" />
    </ol-tile-layer>

    <ol-tile-layer ref="photo1999Layer" title="写真1999年11月" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="tiles/KT991Y-C4-6/{z}/{x}/{y}.png" maxZoom="15" attributions="国土地理院"
        :opaque="false" />
    </ol-tile-layer>

    <ol-tile-layer ref="kanto1988Layer" title="地図1988-2008年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://ktgis.net/kjmapw/kjtilemap/kanto/03/{z}/{x}/{-y}.png"
        maxZoom="15" attributions="今昔マップ on the web" />
    </ol-tile-layer>

    <ol-tile-layer ref="photo1976Layer" title="写真1976年8月" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="tiles/CKT762-C18A-8/{z}/{x}/{y}.png" maxZoom="15" attributions="国土地理院"
        :opaque="false" />
    </ol-tile-layer>

    <ol-tile-layer ref="photo1974Layer" title="写真1974-78年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://cyberjapandata.gsi.go.jp/xyz/gazo1/{z}/{x}/{y}.jpg"
        maxZoom="17" attributions="国土地理院" />
    </ol-tile-layer>

    <ol-tile-layer ref="kanto1972Layer" title="地図1972-82年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://ktgis.net/kjmapw/kjtilemap/kanto/02/{z}/{x}/{-y}.png"
        maxZoom="15" attributions="今昔マップ on the web" />
    </ol-tile-layer>

    <ol-tile-layer ref="photo1969Layer" title="写真1969年10月" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="tiles/KT697Y-C4-6/{z}/{x}/{y}.png" maxZoom="15" attributions="国土地理院"
        :opaque="false" />
    </ol-tile-layer>

    <ol-tile-layer ref="photo1947Layer" title="写真1947年11月">
      <ol-source-xyz crossOrigin="anonymous" url="tiles/USA-M631-A-No2-254/{z}/{x}/{y}.png" maxZoom="15" attributions="国土地理院"
        :opaque="false" />
    </ol-tile-layer>

    <ol-tile-layer ref="kanto1928Layer" title="地図1928-1945年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://ktgis.net/kjmapw/kjtilemap/kanto/01/{z}/{x}/{-y}.png"
        maxZoom="15" attributions="今昔マップ on the web" />
    </ol-tile-layer>

    <ol-tile-layer ref="kanto1894Layer" title="地図1894-1915年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://ktgis.net/kjmapw/kjtilemap/kanto/00/{z}/{x}/{-y}.png"
        maxZoom="15" attributions="今昔マップ on the web" />
    </ol-tile-layer>

    <ol-tile-layer ref="kanto1888Layer" title="地図1888年" :visible="false">
      <ol-source-xyz crossOrigin="anonymous" url="https://mapwarper.h-gis.jp/maps/tile/1006/{z}/{x}/{y}.png"
        maxZoom="17" attributions="スタンフォード大学/日本版MapWarper" :opaque="false" />
    </ol-tile-layer>

    <ol-vector-layer ref="geojsonLayer" title="川場村大字">
      <ol-source-vector :url="url" :format="geoJson">
        <ol-style :overrideStyleFunction="styleFn"></ol-style>
      </ol-source-vector>
    </ol-vector-layer>

    <ol-layerswitcher-control v-if="layerList.length > 0" />
    <ol-fullscreen-control />
    <ol-printdialog-control />
  </ol-map>
</template>

<script setup lang="ts">
import type { Feature } from "ol";
import { Fill, Stroke, Style, Text } from "ol/style";
import { ref, onMounted, inject } from "vue";
import type Map from "ol/Map";
import { transform } from "ol/proj";

const center = ref(transform([139.12983931257054, 36.72759597091191], 'EPSG:4326', 'EPSG:3857'));
const projection = ref("EPSG:3857");
const zoom = ref(13);
const mapRef = ref<{ map: Map }>();
const layerList = ref([] as any[]);
const osmLayer = ref(null);
const reliefLayer = ref(null);
const kanto1888Layer = ref(null);
const kanto1894Layer = ref(null);
const kanto1928Layer = ref(null);
const photo1947Layer = ref(null);
const photo1969Layer = ref(null);
const kanto1972Layer = ref(null);
const photo1974Layer = ref(null);
const photo1976Layer = ref(null);
const kanto1988Layer = ref(null);
const photo1999Layer = ref(null);
const photoJapanLayer = ref(null);
const cyberJapanLayer = ref(null);
const geojsonLayer = ref(null);
const url = ref("./kawaba.geojson");
const format = inject("ol-format");
const geoJson = new format.GeoJSON();
//let map: Map;

function styleFn(feature: Feature) {
  return new Style({
    stroke: new Stroke({
      color: "red",
      width: 2,
    }),
    fill: new Fill({
      color: 'rgba(255, 0, 0, 0.1)',
    }),
    text: new Text({
      textBaseline: 'middle',
      font: "Bold 20px/2 'Open Sans'",
      text: feature.get('S_NAME'),
      fill: new Fill({ color: 'red' }),
      stroke: new Stroke({ color: 'white', width: 2 }),
      placement: 'point',
      overflow: true,
    })
  });
}

onMounted(() => {
  //map = mapRef.value!.map;
  layerList.value.push((osmLayer.value! as any).tileLayer);
  layerList.value.push((photoJapanLayer.value! as any).tileLayer);
  layerList.value.push((cyberJapanLayer.value! as any).tileLayer);
  layerList.value.push((reliefLayer.value! as any).tileLayer);
  layerList.value.push((photo1999Layer.value! as any).tileLayer);
  layerList.value.push((kanto1988Layer.value! as any).tileLayer);
  layerList.value.push((photo1976Layer.value! as any).tileLayer);
  layerList.value.push((photo1974Layer.value! as any).tileLayer);
  layerList.value.push((kanto1972Layer.value! as any).tileLayer);
  layerList.value.push((photo1969Layer.value! as any).tileLayer);
  layerList.value.push((photo1947Layer.value! as any).tileLayer);
  layerList.value.push((kanto1928Layer.value! as any).tileLayer);  
  layerList.value.push((kanto1894Layer.value! as any).tileLayer);
  layerList.value.push((kanto1888Layer.value! as any).tileLayer);
  layerList.value.push((geojsonLayer.value! as any).vectorLayer);
});
</script>

<style scoped>
</style>