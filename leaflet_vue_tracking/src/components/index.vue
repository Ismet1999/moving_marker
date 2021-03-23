<template>
  <l-map :zoom="14" :center="initialLocation" ref="map">
    <l-icon-default />
    <l-tile-layer :url="mapData.url" :attribution="mapData.attribution" />
    <l-moving-marker
      :lat-lng="markerFirst"
      :duration="stateDuration"
      :keep-at-center="true"
      :icon="icon"
    >
    </l-moving-marker>
    <l-polyline
      :lat-lngs="polyline.latlngs"
      :color="polyline.color"
      :fill="false"
    ></l-polyline>
    <l-control>
      <span id=""> {{ Math.ceil(speed * 3.6) }} km/h </span>
      <span id=""> {{ stateDuration }} </span>
      <p id="">{{ markerFirst }}</p>
    </l-control>
    <l-control>
      <button @click="startPause">
        start/pause
      </button>
    </l-control>
  </l-map>
</template>

<script>
import L from 'leaflet'
import {
  LMap,
  LTileLayer,
  LIconDefault,
  LPolyline,
  LControl,
} from 'vue2-leaflet'
import LMovingMarker from '../../lib/index.vue'
import coordinates from '../assets/coordinates.json'
import { coords } from '../assets/coords'

// function rand(n) {
//   const max = n + 0.01;
//   const min = n - 0.01;
//   return Math.random() * (max - min) + min;
// }

// const locations = [];
// for (let i = 0; i < 10; i++) {
//   locations.push({
//     id: i,
//     latlng: L.latLng(48.8929425, 2.3821873),
//     text: "Moving Marker #" + i,
//   });
// }

const icon = L.icon({
  iconUrl:
    'https://s3-eu-west-1.amazonaws.com/ct-documents/emails/A-static.png',
  iconSize: [21, 31],
  iconAnchor: [10.5, 31],
  popupAnchor: [4, -25],
})

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
          'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
      },
      polyline: {
        color: 'green',
        latlngs: coords,
      },
      interval: null,
      markerFirst: null,
      stateDuration: this.duration,
      speed: 0,
      oldDuration: null,
    }
  },
  computed: {
    // get: function() {
    //   return this.stateDuration;
    // },
    // set : function(newValue){
    // }
  },
  watch: {
    stateDuration: {
      handler(value, oldValue) {
        if (value !== oldValue) {
          if (this.playState) {
            clearInterval(this.interval)
            const setRandomLatLng = () => {
              console.log('value !== oldValue', value, oldValue)

              this.stateDuration =
                this.pDate(coordinates[this.countTurn + 1][0]) -
                this.pDate(coordinates[this.countTurn][0])

              this.markerFirst = this.pl(coordinates[this.countTurn])

              // console.log(this.$refs.map.mapObject.distance([41, 69], [35, 87]))
              this.speed =
                this.$refs.map.mapObject.distance(
                  this.pl(coordinates[this.countTurn + 1]),
                  this.pl(coordinates[this.countTurn])
                ) /
                (this.stateDuration / 1000)

              this.countTurn++
              console.log('this.stateDuration ', this.stateDuration)
            }
            this.interval = setInterval(() => {
              if (this.playState) {
                setRandomLatLng()
              }
            }, value)
            setRandomLatLng()
          }
        }
      },
      immediate: true,
    },
  },

  mounted() {
    this.markerFirst = this.pl(coordinates[0])
  },
  methods: {
    pl(value) {
      return {
        lng: value[1],
        lat: value[2],
      }
    },
    pDate(value) {
      return new Date(value)
    },

    startPause() {
      if (this.playState) {
        this.playState = false
        this.oldDuration = this.stateDuration
        this.stateDuration = 0
      } else {
        this.stateDuration = this.oldDuration
        this.playState = true
      }
    },

    // getSpeed(value){

    // }
  },
}
</script>

<style>
@import '~leaflet/dist/leaflet.css';
</style>
