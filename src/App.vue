<template>
  <div>
    <l-map style="height:90vh">
      <l-tile-layer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
      <div v-for="pos in locations">
        <l-marker :lat-lng="[pos.lat, pos.lng]">
          <l-popup>
            {{ new Date(parseInt(pos.ts)).toString().split("(")[0] }}<br>
            GPS accuracy: {{ pos.accuracy }}<br>
            GPS velocity: {{ pos.velocity }}<br>
            Previous position distance: {{ pos.previous_distance }}<br>
            Speed between previous position: {{ pos.previous_distance / ((pos.ts-pos.previous_timestamp)/1000/60/60) }}
          </l-popup>
        </l-marker>
      </div>
    </l-map>
  </div>
</template>

<script>
import { Icon } from 'leaflet';

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

import "leaflet/dist/leaflet.css"
import { LMap, LPopup, LPolyline, LMarker, LTileLayer } from "vue2-leaflet";
const history = require("../history.json");
export default {
  name: 'App',
  components: {
    LMap,
    LPolyline,
    LMarker,
    LPopup,
    LTileLayer
  },
  data() {
    return {
      locations: [],
    }
  },
  methods: {
    distance(lon1, lat1, lon2, lat2) {
        var radlat1 = Math.PI * lat1/180
        var radlat2 = Math.PI * lat2/180
        var theta = lon1-lon2
        var radtheta = Math.PI * theta/180
        var dist = Math.sin(radlat1) * Math.sin(radlat2) + Math.cos(radlat1) * Math.cos(radlat2) * Math.cos(radtheta);
        dist = Math.acos(dist)
        dist = dist * 180/Math.PI
        dist = dist * 60 * 1.1515
        return dist
    }
  },
  mounted() {
    let previous_location = null;
    for(let location of history.locations) {
      if(previous_location != null) this.locations.push({
        lat: location.latitudeE7/1e7,
        lng: location.longitudeE7/1e7,
        ts: location.timestampMs,
        accuracy: location.accuracy,
        velocity: location.velocity,
        previous_distance: this.distance(location.longitudeE7/1e7, location.latitudeE7/1e7, previous_location.longitudeE7/1e7, previous_location.latitudeE7/1e7),
        previous_timestamp: previous_location.timestampMs
      })
      previous_location = location;

    }
  }
}
</script>

<style>
</style>
