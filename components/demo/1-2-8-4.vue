<template>
  <div>
    <div ref="map" class="map"></div>
    <input id="swipe" type="range" style="width: 100%" />
  </div>
</template>

<script>
export default {
  mounted() {
    let {
      Map,
      View,
      layer: { Tile: TileLayer },
      source: { OSM, XYZ },
      render: { getRenderPixel },
    } = ol;

    const osm = new TileLayer({
      source: new OSM(),
    });
    const attributions =
      '<a href="https://www.maptiler.com/copyright/" target="_blank">&copy; MapTiler</a> ' +
      '<a href="https://www.openstreetmap.org/copyright" target="_blank">&copy; OpenStreetMap contributors</a>';
    const aerial = new TileLayer({
      source: new XYZ({
        attributions: attributions,
        url:
          "https://api.maptiler.com/tiles/satellite/{z}/{x}/{y}.jpg?key=" +
          mapkeys.maptiler,
        maxZoom: 20,
      }),
    });
    const map = new Map({
      layers: [osm, aerial],
      target: this.$refs.map,
      view: new View({
        center: [12579156, 3274244],
        zoom: 2,
      }),
    });
    const swipe = document.getElementById("swipe");
    aerial.on("prerender", function (event) {
      const ctx = event.context;
      const mapSize = map.getSize();
      const width = mapSize[0] * (swipe.value / 100);
      const tl = getRenderPixel(event, [width, 0]);
      const tr = getRenderPixel(event, [mapSize[0], 0]);
      const bl = getRenderPixel(event, [width, mapSize[1]]);
      const br = getRenderPixel(event, mapSize);
      ctx.save();
      ctx.beginPath();
      ctx.moveTo(tl[0], tl[1]);
      ctx.lineTo(bl[0], bl[1]);
      ctx.lineTo(br[0], br[1]);
      ctx.lineTo(tr[0], tr[1]);
      ctx.closePath();
      ctx.clip();
    });
    aerial.on("postrender", function (event) {
      const ctx = event.context;
      ctx.restore();
    });
    const listener = function () {
      map.render();
    };
    swipe.addEventListener("input", listener);
    swipe.addEventListener("change", listener);
  },
};
</script>