<template>
  <div id="app">
    <div id="svg"></div>
    <div id="popper"></div>
  </div>
</template>

<script>
import { SVG } from "@svgdotjs/svg.js";
import { createPopper } from "@popperjs/core";

const color1 = "009688";
const color2 = "ffffff";

export default {
  name: "App",
  data() {
    return {
      provinces: {
        // Población de Cuba 2008
        CUB99: 11242628, // Cuba
        CUB1319: 403574, // Cienfuegos
        CUB1320: 86110, // Isla de la Juventud
        CUB1321: 730236, // Pinar del Río
        CUB1322: 465019, // Sancti Spíritus
        CUB1363: 422354, // Ciego de Ávila
        CUB1364: 781605, // Camaguey
        CUB1365: 510863, // Guantánamo
        CUB1366: 834616, // Granma
        CUB1367: 1036885, // Holguín
        CUB1368: 534279, // Las Tunas
        CUB1369: 1044848, // Santiago de Cuba
        CUB1428: 2148132, // La Habana
        CUB1429: 498987, // Artemisa
        CUB1430: 687600, // Matanzas
        CUB1431: 806144, // Villa Clara
        CUB5489: 381342 // Mayabeque
      },
      popperElement: null
    };
  },
  methods: {
    getHex(v) {
      return v.toString(16).padStart(2, "0");
    },
    getColor(ratio) {
      if (!isFinite(ratio)) {
        return "#" + color1;
      }

      const r = Math.ceil(
        parseInt(color1.substring(0, 2), 16) * ratio +
          parseInt(color2.substring(0, 2), 16) * (1 - ratio)
      );
      const g = Math.ceil(
        parseInt(color1.substring(2, 4), 16) * ratio +
          parseInt(color2.substring(2, 4), 16) * (1 - ratio)
      );
      const b = Math.ceil(
        parseInt(color1.substring(4, 6), 16) * ratio +
          parseInt(color2.substring(4, 6), 16) * (1 - ratio)
      );

      return "#" + this.getHex(r) + this.getHex(g) + this.getHex(b);
    },
    loadMap() {
      fetch("/cu.svg")
        .then(r => r.text())
        .then(i => {
          let startOfSvg = i.indexOf("<svg");
          startOfSvg = startOfSvg >= 0 ? startOfSvg : 0;
          const cuba = SVG(i.slice(startOfSvg))
            .addTo(document.getElementById("svg"))
            .size("100%", "500");
          for (const region of cuba.find("path")) {
            const rv = this.provinces[region.id()];
            if (isFinite(rv)) {
              region.fill(this.getColor(rv / 2148132));
            }
            region.on("mouseover", () => {
              this.popperElement.innerText = `${region.attr("name")} : ${rv}`;
              this.popperElement.style.visibility = "visible";
              createPopper(region.node, this.popperElement, {
                placement: "top"
              });
            });

            region.on("mouseleave", () => {
              this.popperElement.style.visibility = "hidden";
            });
          }
        });
    }
  },
  mounted() {
    this.loadMap();
    this.popperElement = document.getElementById("popper");
  }
};
</script>
<style>
path {
  cursor: pointer;
}
path:hover {
  fill: #3f51b5;
}
#popper {
  visibility: hidden;
  padding: 1em;
  background: white;
  border-radius: 2px;
  box-shadow: 0 0 10px 3px rgba(0, 0, 0, 0.3);
}
</style>
