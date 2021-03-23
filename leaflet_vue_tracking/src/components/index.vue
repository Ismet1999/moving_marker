<template>
  <l-map :zoom="18" :center="initialLocation" ref="map">
    <l-icon-default />
    <l-tile-layer :url="mapData.url" :attribution="mapData.attribution" />
    <l-moving-marker
      :lat-lng="markerFirst"
      :duration="stateDuration"
      :keep-at-center="true"
      :icon="icon"
      ref="marker"
    >
    </l-moving-marker>
    <l-polyline
      :lat-lngs="polyline.latlngs"
      :color="polyline.color"
      :fill="false"
    ></l-polyline>
    <l-control>
      <span id=""> {{ Math.ceil(speed * 3.6) }} km/h </span>
    </l-control>
    <l-control>
      <button @click="startPauseMove">
        start/pause
      </button>
    </l-control>
  </l-map>
</template>

<script>
import L from "leaflet";
import {
  LMap,
  LTileLayer,
  LIconDefault,
  LPolyline,
  LControl,
} from "vue2-leaflet";
import LMovingMarker from "../../lib/index.vue";
import coordinates from "../assets/coordinates.json";
import { coords } from "../assets/coords";

const icon = L.icon({
  iconUrl:
    "https://s3-eu-west-1.amazonaws.com/ct-documents/emails/A-static.png",
  iconSize: [21, 31],
  iconAnchor: [10.5, 31],
  popupAnchor: [4, -25],
});

export default {
  components: {
    LMap,
    LTileLayer,
    LIconDefault,
    LPolyline,
    LControl,
    LMovingMarker,
  },
  props: {
    duration: { type: Number, default: 0 },
    keepAtCenter: { type: Boolean, default: false },
  },
  data() {
    return {
      playState: true,
      icon,
      countTurn: 0,
      initialLocation: L.latLng(55.651365, 37.610225),
      mapData: {
        attribution:
          '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        url:
          "https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png",
      },
      polyline: {
        color: "green",
        latlngs: coords,
      },
      interval: null,
      markerFirst: null,
      stateDuration: this.duration,
      speed: 0,

      speedMotion: 1,
      initInterval: null,
      initTimeout: null,
    };
  },
  mounted() {
    this.markerFirst = this.pl(coordinates[0])
  },
  methods: {
    pl(value) {
      return {
        lng: value[1],
        lat: value[2],
      };
    },
    pDate(value) {
      return new Date(value);
    },

    intervalMove() {
      this.stateDuration =
        this.pDate(coordinates[this.countTurn + 1][0]) -
        this.pDate(coordinates[this.countTurn][0]);
      this.speed =
        this.$refs.map.mapObject.distance(
          this.pl(coordinates[this.countTurn + 1]),
          this.pl(coordinates[this.countTurn])
        ) /
        (this.stateDuration / 1000);

      clearInterval(this.initInterval);
      this.initInterval = setInterval(() => {
        this.markerFirst = this.pl(coordinates[this.countTurn]);
        this.countTurn++;
        this.intervalMove();
      }, this.stateDuration);
    },

    startPauseMove() {
      if (this.playState) {
        this.intervalMove();
        this.playState = false;
      } else {
        this.$refs.marker.mapObject.slideCancel();
        clearInterval(this.initInterval);
        this.playState = true;
      }
    },
  },
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";
</style>
