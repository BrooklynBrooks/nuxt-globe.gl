<template>
  <div>
    <!-- <div>
      <h2>1111111111</h2>
    </div> -->
    <div id="globeViz"></div>
  </div>
</template>

<script>
import Globe from 'globe.gl'
import * as d3 from 'd3'
// import * as axios from '@nuxtjs/axios'

export default {
  components: {},
  async asyncData({ $axios }) {
    const url =
      'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_month.geojson'
    const res = await $axios.$get(url)
    return {
      ddd: res
    }
  },
  data() {
    return {
      globe: Globe()
    }
  },
  mounted() {
    const s = document.getElementById('globeViz')
    const weightColor = d3
      .scaleLinear()
      .domain([0, 60])
      .range(['lightblue', 'darkred'])
      .clamp(true)

    this.globe(s)
      .globeImageUrl('//unpkg.com/three-globe/example/img/earth-night.jpg')
      .hexBinPointLat((d) => d.geometry.coordinates[1])
      .hexBinPointLng((d) => d.geometry.coordinates[0])
      .hexBinPointWeight((d) => d.properties.mag)
      .hexAltitude(({ sumWeight }) => sumWeight * 0.0025)
      .hexTopColor((d) => weightColor(d.sumWeight))
      .hexSideColor((d) => weightColor(d.sumWeight))
      .hexLabel(
        (d) => `
        <b>${d.points.length}</b> earthquakes in the past month:<ul><li>
          ${d.points
            .slice()
            .sort((a, b) => b.properties.mag - a.properties.mag)
            .map((d) => d.properties.title)
            .join('</li><li>')}
        </li></ul>
      `
      )
    this.globe.hexBinPointsData(this.ddd.features)
  }
}
</script>
