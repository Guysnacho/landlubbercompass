<template>
  <v-app id="inspire">
    <v-app-bar app color="primary" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title class="titles">LandLubber Compass</v-toolbar-title>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" app>
      <v-list dense>
        <v-list-item link>
          <v-list-item-action>
            <v-icon>mdi-fish</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Catches</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item link>
          <v-list-item-action>
            <v-icon>mdi-anchor</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Bounties in your area</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container class="fill-height" fluid>
        <v-overlay v-model="dialog">{{}}</v-overlay>
        <l-map
          style="height: 100%; width: 100%"
          :zoom="zoom"
          :center="center"
          @update:zoom="zoomUpdated"
          @update:center="centerUpdated"
          @update:bounds="boundsUpdated"
        >
          <l-tile-layer :url="url"></l-tile-layer>
          <l-control position="bottomright">
            <v-text-field
              outlined
              placeholder="Enter a sea-dweller"
              class="pb-0"
              v-model="animal"
            ></v-text-field>
            <v-btn dark @click="query(animal)">Search the Seas</v-btn>
          </l-control>
        </l-map>
      </v-container>
    </v-main>
    <v-footer color="primary" app>
      <span class="white--text">&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import { LMap, LTileLayer, LControl } from "vue2-leaflet";
import wiki from "wikijs";
export default {
  data: () => ({
    drawer: null,
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
    zoom: 3,
    center: [10, 10],
    bounds: null,
    animal: undefined,
    dialog: false,
    searchedData: undefined
  }),

  methods: {
    zoomUpdated(zoom) {
      this.zoom = zoom;
    },
    centerUpdated(center) {
      this.center = center;
    },
    boundsUpdated(bounds) {
      this.bounds = bounds;
    },
    query(animal) {
      if (animal) {
        wiki()
          .page(animal)
          .then(
            page =>
              (this.searchedData = {
                summary: page.summary(),
                title: page.info()
              })
          );
      } else {
        alert("You didn't enter a sea lubber me matey!");
      }
    }
  },

  components: {
    LMap,
    LTileLayer,
    LControl
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Amatic+SC:wght@700&family=Sarala&display=swap");
body {
  font-family: "Sarala", sans-serif;
}

.titles {
  font-family: "Amatic SC", cursive;
}
</style>
