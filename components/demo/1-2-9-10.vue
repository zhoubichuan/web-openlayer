<template>
  <div id="map" class="map">
    <div id="popup"></div>
  </div>
</template>

<script>
export default {
  mounted() {
    let {
      Feature,
      Overlay,
      geom: { Point },
      Map,
      View,
      layer: { Tile: TileLayer, Vector: VectorLayer },
      source: { TileJSON, Vector: VectorSource },
      style: { Icon, Style, Text },
      proj: { fromLonLat },
      render: { getVectorContext },
    } = ol
    const rasterLayer = new TileLayer({
      source: new TileJSON({
        url:
          "https://a.tiles.mapbox.com/v4/aj.1x1-degrees.json?secure=1&access_token=" +
          mapkeys.mapbox,
        crossOrigin: "anonymous",
      }),
    })
    const iconFeature = new Feature({
      geometry: new Point(fromLonLat([0, -10])),
      name: "Fish.1",
    })
    const feature1 = new Feature({
      geometry: new Point(fromLonLat([0, -10])),
      name: "Fish.1 Island",
    })
    const feature2 = new Feature({
      geometry: new Point(fromLonLat([-30, 10])),
      name: "Fish.2 Island",
    })
    const iconStyle = new Style({
      image: new Icon({
        anchor: [0.5, 0.9],
        src: this.$withBase("/data/fish.png"),
        crossOrigin: "",
        scale: [0, 0],
        rotation: Math.PI / 4,
      }),
      text: new Text({
        text: "FISH\nTEXT",
        scale: [0, 0],
        rotation: Math.PI / 4,
        textAlign: "center",
        textBaseline: "top",
      }),
    })
    let i = 0
    let j = 45
    iconFeature.setStyle(function () {
      const x = Math.sin((i * Math.PI) / 180) * 3
      const y = Math.sin((j * Math.PI) / 180) * 4
      iconStyle.getImage().setScale([x, y])
      iconStyle.getText().setScale([x, y])
      return iconStyle
    })
    rasterLayer.on("postrender", function (event) {
      const vectorContext = getVectorContext(event)
      const x = Math.cos((i * Math.PI) / 180) * 3
      const y = Math.cos((j * Math.PI) / 180) * 4
      iconStyle.getImage().setScale([x, y])
      iconStyle.getText().setScale([x, y])
      vectorContext.drawFeature(feature2, iconStyle)
    })
    const vectorSource = new VectorSource({
      features: [iconFeature, feature1, feature2],
    })
    const vectorLayer = new VectorLayer({
      source: vectorSource,
    })
    const map = new Map({
      layers: [rasterLayer, vectorLayer],
      target: this.$refs.map,
      view: new View({
        center: fromLonLat([-15, 0]),
        zoom: 3,
      }),
    })
    setInterval(function () {
      i = (i + 4) % 360
      j = (j + 5) % 360
      vectorSource.changed()
    }, 1000)
    const element = document.getElementById("popup")
    const popup = new Overlay({
      element: element,
      positioning: "bottom-center",
      stopEvent: false,
    })
    map.addOverlay(popup)
    map.on("click", function (evt) {
      const feature = map.forEachFeatureAtPixel(evt.pixel, function (feature) {
        return feature
      })
      $(element).popover("dispose")
      if (feature) {
        popup.setPosition(evt.coordinate)
        $(element).popover({
          placement: "top",
          html: true,
          animation: false,
          content: feature.get("name"),
        })
        $(element).popover("show")
      }
    })
    map.on("pointermove", function (e) {
      const pixel = map.getEventPixel(e.originalEvent)
      const hit = map.hasFeatureAtPixel(pixel)
      map.getTarget().style.cursor = hit ? "pointer" : ""
    })
    map.on("movestart", function () {
      $(element).popover("dispose")
    })
  },
}
</script>