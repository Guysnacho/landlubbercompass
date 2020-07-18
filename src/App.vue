<template>
  <v-app id="inspire">
    <v-main>
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
      <v-container class="fill-height" fluid>
        <v-overlay v-model="dialog">{{}}</v-overlay>
        <l-map
          ref="map"
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
              @submit="query(animal)"
            ></v-text-field>
            <v-btn dark @click="query(animal)">Search the Seas</v-btn>
          </l-control>
          <l-marker
            v-if="searchedData.locations[0]"
            :lat-lng="[results[0].y, results[0].x]"
          >
            <l-popup>
              <v-card style="height: 75px;">
                <v-img src="searchedData.images"></v-img>
                <v-list>
                  <v-list-item two-line>
                    <v-list-item-content>
                      <v-list-item-title>{{
                        searchedData.title
                      }}</v-list-item-title>
                      <v-list-item-subtitle>{{
                        searchedData.summary
                      }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card>
            </l-popup>
          </l-marker>
        </l-map>
      </v-container>
    </v-main>
    <v-footer color="primary" app>
      <span class="white--text">&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import { LMap, LTileLayer, LControl, LMarker, LPopup } from "vue2-leaflet";
import { OpenStreetMapProvider } from "leaflet-geosearch";
import axios from "axios";
import wiki from "wikijs";
export default {
  data: () => ({
    drawer: null,
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
    provider: new OpenStreetMapProvider(),
    zoom: 3,
    center: [10, 10],
    bounds: null,
    animal: undefined,
    dialog: false,
    results: [{ x: 0, y: 0 }],
    searchedData: { title: "", summary: "", images: "", locations: [] }
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
          .then(page => {
            page.info().then(Response => {
              this.searchedData.title = Response.imageCaption;
            });
            page.summary().then(Response => {
              this.searchedData.summary = Response;
            });
            page.images().then(Response => {
              this.searchedData.images = Response[0];
            });
          })
          .finally(() => {
            axios({
              method: "get",
              url:
                "https://fishbase.ropensci.org/ecosystem?fields=" +
                this.searchedData.title.trim()
            }).then(Response => {
              this.scrapeEcosystem(Response.data.data);
            });
          });
      } else {
        alert("You didn't enter a sea lubber me matey!");
      }
    },
    scrapeEcosystem(raw) {
      console.log(raw);
      //TODO - I want to go through each one and get their ecosystem name and type
      let temp = [];
      raw.forEach(element => {
        temp = [...temp, element.EcosystemName + " " + element.EcosystemType];
      });
      temp = Array.from(new Set(temp));
      this.searchedData.locations = temp;
      this.getCoords(temp[0]);
    },
    async getCoords(args) {
      this.results = await this.provider.search({ query: args });
    }
  },

  components: {
    LMap,
    LTileLayer,
    LControl,
    LMarker,
    LPopup
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Amatic+SC:wght@700&family=Sarala&display=swap");
@import url("https://unpkg.com/leaflet-geosearch@2.6.0/assets/css/leaflet.css");
body {
  font-family: "Sarala", sans-serif;
}

.titles {
  font-family: "Amatic SC", cursive;
}
</style>
