<template>
  <div ref="map" class="map" @mapMounted="mapMounted"></div>
</template>

<script>
let { Map, View, extent, proj, tilegrid, layer, source } = ol;
let urls = [
  "http://t0.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
  // "http://t0.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
  // "http://t0.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
  // "http://t0.tianditu.com/DataServer?T=cia_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
  // "http://t0.tianditu.com/DataServer?T=ter_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
  // "http://t0.tianditu.com/DataServer?T=cta_w&x={x}&y={y}&l={z}&tk=d97ee4980a986e7d0c4f0a8c8f103a94",
];
let mapInstance;
export default {
  name: "WebOpenlayers2",
  props: {
    view: {
      type: Object,
      default: () => ({
        center() {
          return [12579156, 3274244];
        },
        zoom: 12,
      }),
    },
    layer: {
      type: Array,
      default: () =>
        urls.map((item) => ({
          name: "Tile",
          source: {
            name: "XYZ",
            config() {
              return {
                url: item,
                crossOrigin: "anonymous",
              };
            },
          },
        })),
    },
  },
  mounted() {
    const projection = proj.get("EPSG:900913");
    const projectionExtent = projection.getExtent();
    const size = extent.getWidth(projectionExtent) / 256;
    const resolutions = new Array(19);
    const matrixIds = new Array(19);
    for (let z = 0; z < 19; ++z) {
      resolutions[z] = size / Math.pow(2, z);
      matrixIds[z] = z;
    }
    const tileGrid = new tilegrid.WMTS({
      origin: extent.getTopLeft(projectionExtent),
      resolutions: resolutions,
      matrixIds: matrixIds,
    });
    let target = this.$refs.map;
    this.$emit("mapTarget", target);
    let layers = this.layer.map(
      (i) =>
        new layer[i.name]({
          source: new source[i.source.name](
            i.source.config && i.source.config({ projection, tileGrid })
          ),
        })
    );
    this.$emit("mapLayers", layers);
    let view = new View({
      ...this.view,
      center:
        typeof this.view.center === "function"
          ? this.view.center(proj)
          : this.view.center,
    });
    this.$emit("mapView", view);
    mapInstance = new Map({
      target,
      layers,
      view,
    });
    this.$emit("mapMounted", mapInstance);
  },
  methods: {
    mapMounted() {
      this.$emit("mapMounted", mapInstance);
    },
    getInstance() {
      return mapInstance;
    },
    getLayer(){

    },
    getSource(){

    }
  },
};
</script>
<style scoped>
.map {
  width: 100%;
  height: 100%;
}
</style>

