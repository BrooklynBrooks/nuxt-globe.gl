<template>
  <div>
    <div :id="globeVisId"></div>
  </div>
</template>

<script>
import Globe from 'globe.gl'
import * as d3 from 'd3'

export default {
  components: {},
  async asyncData({ $axios }) {
    const url =
      'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_month.geojson'
    const res = await $axios.$get(url)
    return {
      geoData: res
    }
  },
  data() {
    return {
      globe: Globe(),
      // globeImageUrl: '//unpkg.com/three-globe/example/img/earth-dark.jpg',
      globeImageUrl: '../hoge.png',
      globeVisId: 'globeViz',
      backgroundColor: '#e5e5e5'
    }
  },
  mounted() {
    const globeElem = document.getElementById(this.globeVisId)
    const weightColor = d3
      .scaleLinear()
      .domain([0, 60])
      .range(['red', 'yellow'])
      .clamp(true)

    this.globe(globeElem)
      .globeImageUrl(this.globeImageUrl)
      .backgroundColor(this.backgroundColor)
      .hexBinPointLat((d) => d.geometry.coordinates[1])
      .hexBinPointLng((d) => d.geometry.coordinates[0])
      .hexBinPointWeight((d) => d.properties.mag)
      .hexAltitude(({ sumWeight }) => sumWeight * 0.001)
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
    this.globe.hexBinPointsData(this.geoData.features)
  }
}
</script>
