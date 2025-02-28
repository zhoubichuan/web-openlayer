<template>
  <div>
    <div ref="map" class="map"></div>
    <div>
      <h5>对比度拉伸</h5>
      <ul>
        <li>
          Min <input type="range" min="1000" max="10000" ref="inputmin" />
          <span ref="outputmin"></span>
        </li>
        <li>
          Max <input type="range" min="10000" max="50000" ref="inputmax" />
          <span ref="outputmax"></span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  async mounted() {
    let {
      Map,
      View,
      layer: { WebGLTile: TileLayer },
      source: { DataTile: DataTileSource },
      proj: { fromLonLat },
    } = ol;
    const COG =
      "https://storage.googleapis.com/open-cogs/stac-examples/20201211_223832_CS2_analytic.tif";

    function numpyTileLoader(z, x, y) {
      const url = `https://api.cogeo.xyz/cog/tiles/WebMercatorQuad/${z}/${x}/${y}@1x`;
      return fetch(`${url}?format=npy&url=${encodeURIComponent(COG)}`)
        .then((r) => r.arrayBuffer())
        .then((buffer) => NumpyLoader.fromArrayBuffer(buffer))
        .then((numpyData) => {
          const dataTile = new Float32Array(256 * 256 * 5);
          const bandSize = 256 * 256;
          for (let x = 0; x < 256; x++) {
            for (let y = 0; y < 256; y++) {
              const px = x + y * 256;
              dataTile[px * 5 + 0] = numpyData.data[y * 256 + x];
              dataTile[px * 5 + 1] = numpyData.data[bandSize + y * 256 + x];
              dataTile[px * 5 + 2] = numpyData.data[bandSize * 2 + y * 256 + x];
              dataTile[px * 5 + 3] = numpyData.data[bandSize * 3 + y * 256 + x];
              dataTile[px * 5 + 4] =
                numpyData.data[bandSize * 4 + y * 256 + x] > 0 ? 1.0 : 0;
            }
          }
          return dataTile;
        });
    }

    const interpolateBand = (bandIdx) => [
      "interpolate",
      ["linear"],
      ["band", bandIdx],
      ["var", "bMin"],
      0,
      ["var", "bMax"],
      1,
    ];
    const initialMin = 3000;
    const initialMax = 18000;
    const numpyLayer = new TileLayer({
      style: {
        color: [
          "array",
          interpolateBand(3),
          interpolateBand(2),
          interpolateBand(1),
          ["band", 5],
        ],
        variables: {
          bMin: initialMin,
          bMax: initialMax,
        },
      },
      source: new DataTileSource({
        loader: numpyTileLoader,
        bandCount: 5,
      }),
    });
    const map = new Map({
      target: this.$refs.map,
      layers: [numpyLayer],
      view: new View({
        center: fromLonLat([172.933, 1.3567]),
        zoom: 15,
      }),
    });
    const inputMin = this.$refs.inputmin;
    const inputMax = this.$refs.inputmax;
    const outputMin = this.$refs.outputmin;
    const outputMax = this.$refs.outputmax;
    const handleMin = (evt) => {
      numpyLayer.updateStyleVariables({
        bMin: parseFloat(evt.target.value),
        bMax: parseFloat(inputMax.value),
      });
      outputMin.innerText = evt.target.value;
    };
    inputMin.addEventListener("input", handleMin);
    inputMin.addEventListener("change", handleMin);
    const handleMax = (evt) => {
      numpyLayer.updateStyleVariables({
        bMin: parseFloat(inputMin.value),
        bMax: parseFloat(evt.target.value),
      });
      outputMax.innerText = evt.target.value;
    };
    inputMax.addEventListener("input", handleMax);
    inputMax.addEventListener("change", handleMax);
    inputMin.value = initialMin;
    inputMax.value = initialMax;
    outputMin.innerText = initialMin;
    outputMax.innerText = initialMax;
  },
};
</script>