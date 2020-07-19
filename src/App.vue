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
        </v-list>
      </v-navigation-drawer>
      <v-container class="fill-height" fluid>
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
              @blur="getControl()"
            ></v-text-field>
            <v-btn
              dark
              @click="
                marker = false;
                query(animal);
                findAquarium();
              "
              >Search the Seas</v-btn
            >
          </l-control>
          <l-marker v-if="marker" :lat-lng="[results[0], results[1]]">
            <l-popup style="width: 320px" :options="options">
              <v-carousel
                continuous
                hide-delimiter-background
                show-arrows-on-hover
                class="ml--1"
                height="300"
              >
                <v-carousel-item>
                  <v-row justify="center">
                    <v-img
                      :src="searchedData.images"
                      max-width="75%"
                      class="justify-center"
                    ></v-img>
                  </v-row>
                  <v-card-text class="carouseltext text-center">
                    {{ searchedData.title }} -
                    {{ "\n" + searchedData.summary }}
                  </v-card-text>
                </v-carousel-item>
                <v-carousel-item>
                  <v-card-title class="carouseltext text-center"
                    >An aquarium near you!</v-card-title
                  >
                  <v-card-text>{{ results[3] }}</v-card-text>
                  <v-card-text>{{ results[2] }}</v-card-text>
                </v-carousel-item>
              </v-carousel>
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
import { GoogleProvider } from "leaflet-geosearch";
import axios from "axios";
import wiki from "wikijs";
export default {
  data: () => ({
    drawer: false,
    url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
    provider: new GoogleProvider({
      params: {
        key: "AIzaSyBtogduRnR_iuKtj8IFJ8fB41bhffv4Wcw"
      }
    }),
    center: [10, 10],
    zoom: 3,
    marker: false,
    animal: undefined,
    city: "",
    results: {},
    control: [],
    searchedData: { title: "", summary: "", images: "", locations: [] },
    options: {
      autoClose: false,
      closeButton: true
    }
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
            this.searchedData.title = page.raw.title;
            page.summary().then(Response => {
              this.searchedData.summary = Response.substring(
                0,
                Response.indexOf(".")
              );
            });
            page.images().then(Response => {
              for (var x = 0; x < Response.length; x++) {
                let temp = Response[x].toUpperCase();
                if (
                  temp.includes(
                    this.animal
                      .slice(0, this.animal.indexOf(" ") + 1)
                      .toUpperCase()
                  )
                ) {
                  this.searchedData.images = Response[x];
                  break;
                }
              }
            });
          });
      } else {
        alert("You didn't enter a sea lubber me matey!");
      }
    },
    getControl() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(Response => {
          this.control = [Response.coords.latitude, Response.coords.longitude];
          console.log(Response);
          axios({
            method: "get",
            url:
              "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
              this.control[0] +
              "," +
              this.control[1] +
              "&key=AIzaSyBtogduRnR_iuKtj8IFJ8fB41bhffv4Wcw"
          }).then(Response => {
            this.city = Response.results[0].address_components[3];
          });
        });
      }
    },
    findAquarium() {
      console.log("Populate results");

      this.provider
        .search({ query: "Aquarium" })
        .then(Response => {
          console.log(Response);
          this.results = {
            lat: Response[0].geometry.location.lat,
            lng: Response[0].geometry.location.lng,
            addy: Response[0].formatted_address,
            label: Response[0].name
          };
        })
        .catch(err => {
          console.log(err);
        });
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

.carouseltext {
  color: black;
  font-family: "Sarala", sans-serif;
}
</style>
