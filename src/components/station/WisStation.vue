<template id="wis-station">
  <div class="wis-station" style="display: none"></div>
</template>

<script>
import { circleMarker, geoJSON } from "leaflet/dist/leaflet-src.esm";
import { MarkerClusterGroup } from "leaflet.markercluster/dist/leaflet.markercluster-src.js";

import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";

import { defineComponent } from "vue";

export default defineComponent({
  name: "WisStation",
  template: "#wis-station",
  props: ["features", "map"],
  data: function () {
    return {
      features_: this.features,
      geojsonOptions: {
        onEachFeature: this.onEachFeature,
        pointToLayer: this.pointToLayer,
      },
    };
  },
  mounted: function () {
    this.$nextTick(() => {
      this.onReady()
    })
  },
  methods: {
    onReady() {
      var markerCluster = new MarkerClusterGroup({
        disableClusteringAtZoom: 9,
        chunkedLoading: true,
        chunkInterval: 500,
      });
      markerCluster.addLayer(new geoJSON(this.stations, this.geojsonOptions));
      this.map.addLayer(markerCluster);
    },
    mapClick(e) {
      this.features_.station = e.target.feature;
      this.features_.datastreams.length = 0;
      this.$root.toggleDialog();
      e.originalEvent.stopPropagation();
    },
    onEachFeature(feature, layer) {
      var self = this;
      layer.on("mouseover", function (e) {
        self.features_.station = feature;
        layer.bindPopup(feature.properties.name).openPopup(e.latLng);
      });
      layer.on({
        click: this.mapClick
      });
    },
    pointToLayer(feature, latLng) {
      let fillColor;
      let color;
      let hits = feature.properties.num_obs;
      if (hits === 0) {
        fillColor = "#708090";
        color = "#2E343B";
      } else if (hits <= 7) {
        fillColor = "#FF3300";
        color = "#801A00";
      } else if (hits <= 19) {
        fillColor = "#FF9900";
        color = "#804D00";
      } else {
        fillColor = "#009900";
        color = "#004D00";
      }
      const markerStyle = {
        radius: 4,
        fillColor: fillColor,
        color: color,
        weight: 1,
        opacity: 1,
        fillOpacity: 0.8,
      };
      return circleMarker(latLng, markerStyle);
    },
  },
  computed: {
    stations: function () {
      return this.features_.stations;
    },
  },
});
</script>
