<template>
  <div>
    <div v-if="loading" class="loading">Loading...</div>

    <b-row>
      <b-col cols="3" id="sidebarArea">
        <b-row id="searchArea">
          <b-form inline>
            <b-col cols="8">
              <vue-google-autocomplete
                ref="address"
                id="searchBox"
                class="form-control"
                placeholder="Please type your address"
                country="th"
                v-model="searchAddressInput"
                v-on:placechanged="getAddressData"
              ></vue-google-autocomplete>
            </b-col>
            <b-col cols="4">
              <b-button variant="outline-primary" @click="searchLocation">ค้นหา</b-button>
            </b-col>
          </b-form>
        </b-row>
        <div v-for="(restaurant) in restaurantList" v-bind:key="restaurant.name">
          <b-list-group>
            <b-list-group-item
              v-on:click="goToMarker(restaurant.name, restaurant.location)"
              style="cursor: pointer;"
            >{{restaurant.name}}</b-list-group-item>
          </b-list-group>
        </div>
      </b-col>
      <b-col>
        <div id="map"></div>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import VueGoogleAutocomplete from "vue-google-autocomplete";

let map;
let service;
let infowindow;

export default {
  components: { VueGoogleAutocomplete },
  props: {},

  data() {
    return {
      loading: false,
      currentLocation: { lat: 0, lng: 0 },
      searchAddressInput: "Bangsue",
      restaurantList: []
    };
  },
  mounted() {
    this.$refs.address.focus();
    this.getLocationFromGoogleMapsService(this.searchAddressInput);
  },
  methods: {
    getAddressData: function(addressData, placeResultData, id) {
      this.searchAddressInput = placeResultData.name;
    },
    searchLocation() {
      this.getLocationFromGoogleMapsService(this.searchAddressInput);
    },
    getLocationFromGoogleMapsService() {
      console.log("getLocationFromGoogleMapsService");
      console.log(this.searchAddressInput);
      this.loading = true;
      const url = "http://localhost:8081/restaurant";
      let data = {
        address: this.searchAddressInput
      };
      let postData = {
        method: "POST",
        body: JSON.stringify(data)
      };

      fetch(url, postData)
        .then(response => response.json())
        .then(json => {
          this.loading = false;
          const results = json.results[0];
          const geometry = results.geometry;
          const location = geometry.location;
          const viewport = geometry.viewport;
          this.currentLocation.lat = geometry.location.lat;
          this.currentLocation.lng = geometry.location.lng;

          const geocoder = new google.maps.Geocoder();
          map = new google.maps.Map(document.getElementById("map"), {
            center: this.currentLocation,
            scrollwheel: false,
            zoom: 15
          });
          
          map.setCenter(location);

          infowindow = new google.maps.InfoWindow();
          service = new google.maps.places.PlacesService(map);

          service.nearbySearch(
            {
              location: this.currentLocation,
              radius: 1000,
              type: ["restaurant"]
            },
            (results, status) => {
              if (status === "OK") {
                this.restaurantList = [];
                for (var i = 0; i < results.length; i++) {
                  this.restaurantList.push({
                    name: results[i].name,
                    location: results[i].geometry.location
                  });
                  this.createMarker(results[i]);
                }
              }
            }
          );
        })
        .catch(err => console.error(err));
    },
    createMarker(place) {
      var marker = new google.maps.Marker({
        map: map,
        position: place.geometry.location
      });

      google.maps.event.addListener(marker, "click", function() {
        infowindow.setContent(place.name);
        infowindow.open(map, this);
      });
    },
    goToMarker(name, location) {
      var marker = new google.maps.Marker({
        map: map,
        position: location
      });

      map.setCenter(location);
      infowindow.setContent(name);
      infowindow.open(map, marker);
    }
  }
};
</script>
<style scoped>
#map {
  height: 750px;
  /* height: 100%; */
  width: 100%;
}
#sidebarArea {
  margin-left: 15px;
  /* height: 100%; */
  height: 750px;
  overflow: auto;
}
#searchArea {
  margin: 10px;
}
#listArea {
  margin: 15px;
}
</style>