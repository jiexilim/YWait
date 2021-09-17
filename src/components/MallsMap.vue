<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div class="card">
      <div class="card-body">
        <kendo-map
          :center="[1.3051315545985274, 103.83224131197983]"
          :zoom="12"
          style="width: 100%; height: 470px; margin-left: 35px"
        >
          <kendo-map-layer
            :type="'tile'"
            :url-template="'https://#= subdomain #.tile.openstreetmap.org/#= zoom #/#= x #/#= y #.png'"
            :subdomains="['a', 'b', 'c']"
            :attribution="'&copy; <a href=\'https://osm.org/copyright\'> OpenStreetMap contributors</a>'"
          >

          // ION Orchard
          </kendo-map-layer>
          <kendo-map-marker :location="[1.3041594102285439, 103.83223014972324]">
          </kendo-map-marker>

          // Jewel Changi Airport
          <kendo-map-marker
            :location="[1.3604823693988464, 103.98973855951958]"
          >
          </kendo-map-marker>

          // Westgate
          <kendo-map-marker :location="[1.334732192769414, 103.74266480212091]">
          </kendo-map-marker>

          // VivoCity
          <kendo-map-marker
            :location="[1.2646123613498939, 103.8221963711645]"
          >
          </kendo-map-marker>

          // Bugis Junction
          <kendo-map-marker
            :location="[1.299140962845494, 103.85543758280939]"
          >
          </kendo-map-marker>

          // Suntec City
          <kendo-map-marker
            :location="[1.295360041820001, 103.85918464574937]"
          >
          </kendo-map-marker>
        </kendo-map>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Map",
  props: {
    msg: String,
  },
};
</script>

<style scoped>
.card-body {
  width: 95%;
}
</style>
