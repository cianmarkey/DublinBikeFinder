<template>
<div id="app">
    <!-- <p v-if="walkingDuration">You could walk there in {{walkingDuration}}.</p> -->
    <div v-if="!positionsFilled" id="inputs">
      <AddressField v-on:update:position="user.position = $event"
                    v-on:update:name="user.name = $event"
                    v-on:update:formatted_address="user.formatted_address = $event"
                    v-on:updateDistToStation="updateDistToStation()"
                    fieldPlaceholder="Where you at?"
                    ></AddressField>
      <AddressField v-on:update:position="dest.position = $event"
                    v-on:update:name="dest.name = $event"
                    v-on:update:formatted_address="dest.formatted_address = $event"
                    v-on:updateDistToStation="updateDistToStation()"
                    fieldPlaceholder="Where are ya headed?"
                    ></AddressField>
    </div>
    <div class="filled" v-else>
      <p>From:</p>
      <h3>{{user.formatted_address}}</h3>
      <p>To:</p>
      <h3>{{dest.formatted_address}}</h3>
    </div>
    <div id="results">
      <div v-if="positionsFilled" class="stations">
        <div v-if="stations.length > 0">
          <h2 v-if="positionsFilled && selectedUserStation === null">Pick a station close to you.</h2>
          <transition-group name="fade" tag="">
            <StationListing v-for="station in orderedUserStations"
                            v-on:report-selected="selectedUserStation = $event"
                            :key="station.number"
                            :id="station.number"
                            :name="station.name"
                            :userName="user.name"
                            :position="station.position"
                            :available_bikes="station.available_bikes"
                            :available_bike_stands="station.available_bike_stands"
                            :selected.sync="station.selected"
                            :distToUser="station.distToUser"
                            :wantBike="true"
                            ></StationListing>
          </transition-group>
          <button v-if="stationsToDisplayUser < stations.length && selectedUserStation === null"
                  @click="stationsToDisplayUser += 5"
                  >Show More</button>
        </div>
      </div>
      <div class="stations">
        <div v-if="stations.length > 0 && selectedUserStation !== null">
          <h2 v-if="positionsFilled && selectedDestStation === null">Pick a station close to your destination.</h2>
          <StationListing v-for="station in orderedDestStations"
                          v-on:report-selected="selectedDestStation = $event"
                          :key="station.number + 200"
                          :id="station.number + 200"
                          :name="station.name"
                          :destName="dest.name"
                          :position="station.position"
                          :available_bikes="station.available_bikes"
                          :available_bike_stands="station.available_bike_stands"
                          :selected.sync="station.selected"
                          :distToDest="station.distToDest"
                          :wantBike="false"
                          ></StationListing>
          <button v-if="stationsToDisplayDest < stations.length && selectedDestStation === null"
                    @click="stationsToDisplayDest += 5"
                    >Show More</button>
        </div>
      </div>
      <div id="link" v-if="googleMapLink !== ''">
        <a  :href="googleMapLink">Take me to Google Maps</a>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import _ from "lodash";
import AddressField from "./components/AddressField";
import StationListing from "./components/StationListing";

export default {
  //  Dublin Bikes API Key
  //  f07f12786ba132c30596108298be8624fca5a48a
  name: "App",
  components: {
    AddressField,
    StationListing
  },
  methods: {
    degreeToRadian: function(deg) {
      return deg * Math.PI / 180;
    },
    updateDistToStation: function() {
      if (this.positionsFilled) {
          for (let i = 0; i < this.stations.length; i++) {
            let d1 = this.calcDistance(
              this.user.position.lat,
              this.user.position.lng,
              this.stations[i].position.lat,
              this.stations[i].position.lng
            );
            let d2 = this.calcDistance(
              this.dest.position.lat,
              this.dest.position.lng,
              this.stations[i].position.lat,
              this.stations[i].position.lng
            );
            let t1 = Math.ceil(d1 / this.kmPerMin); //approx travel time walking to station
            let t2 = Math.ceil(d2 / this.kmPerMin); //approx travel time walking from station to dest

            //  Vue's way of adding a property to an object so it gains reactivity.
            this.$set(this.stations[i], "distToUser", t1);
            this.$set(this.stations[i], "distToDest", t2);
            //  Adding a "selected" boolean which will by default be false
            this.$set(this.stations[i], "selected", false);

            //  The user has not picked any stations yet.
            this.selectedUserStation = null;
            this.selectedDestStation = null;
          }
      }
    },
    calcDistance: function(lat1, lng1, lat2, lng2) {
      var d = 0, //  Distance
        r = 6364; //  Radius of Earth in Ireland.

      lat1 = this.degreeToRadian(lat1);
      lng1 = this.degreeToRadian(lng1);
      lat2 = this.degreeToRadian(lat2);
      lng2 = this.degreeToRadian(lng2);

      d =
        2 *
        r *
        Math.asin(
          Math.sqrt(
            Math.sin((lat2 - lat1) / 2) * Math.sin((lat2 - lat1) / 2) +
              Math.cos(lat1) *
                Math.cos(lat2) *
                Math.sin((lng2 - lng1) / 2) *
                Math.sin((lng2 - lng1) / 2)
          )
        );

      return d;
    },
    generateMapLink: function(origin, destination) {
      return (
        "https://www.google.com/maps/dir/?api=1&origin=" +
        origin.lat +
        "," +
        origin.lng +
        "&destination=" +
        destination.lat +
        "," +
        destination.lng +
        "&travelmode=bicycling"
      );
    },
    duration: function(user, dest, modeOfTravel) {
      var answer;

      var origin = new google.maps.LatLng(user.position.lat, user.position.lng),
        destination = new google.maps.LatLng(
          dest.position.lat,
          dest.position.lng
        );

      var service = new google.maps.DistanceMatrixService();
      service.getDistanceMatrix(
        {
          origins: [origin],
          destinations: [destination],
          travelMode: modeOfTravel,
          unitSystem: google.maps.UnitSystem.METRIC
        },
        this.distanceCallback
      );

    },
    distanceCallback: function (response, status) {
        if (status === "OK") {
          this.walkingDuration = response.rows[0].elements[0].duration.text;
          /* var origins = response.originAddresses;
          var destinations = response.destinationAddresses;

          for (var i = 0; i < origins.length; i++) {
            var results = response.rows[i].elements;
            for (var j = 0; j < results.length; j++) {
              var element = results[j];
              var distance = element.distance.text;
              var duration = element.duration.text;
              var from = origins[i];
              var to = destinations[j];
            }
          } */
         //console.log(answer)
         //return answer
        }
      }
  },
  computed: {
    positionsFilled: function() {
      return this.user.position !== null && this.dest.position !== null ? true : false
    },
    /*durationWalking() {
      if (this.selectedUserStation !== null && this.selectedDestStation !== null) {
        return this.duration(this.user, this.dest,'WALKING');
      } else {
        return "hasn't worked yet"
      }
    },*/
    orderedUserStations() {
      var filteredStations = [];
      filteredStations = _.filter(this.stations, function(station) {
        return station.available_bikes > 0;
      });

      var sortedStations = _.orderBy(
        filteredStations,
        ["distToUser", "available_bikes"],
        ["asc", "desc"]
      );

      sortedStations = sortedStations.slice(0,this.stationsToDisplayUser)

      var selectedStation = sortedStations.find(
        station => station.number === this.selectedUserStation
      );
      return selectedStation ? [selectedStation] : sortedStations;
    },
    orderedDestStations() {
      let filteredStations = _.filter(this.stations, station => {
        return (
          station.available_bike_stands > 0 &&
          station.number !== this.selectedUserStation
        );
      });
      var sortedStations = _.orderBy(
        filteredStations,
        ["distToDest", "available_bike_stands"],
        ["asc", "desc"]
      );

      sortedStations = sortedStations.slice(0,this.stationsToDisplayDest)

      var selectedStation = sortedStations.find(
        station => station.number === this.selectedDestStation - 200
      );
      return selectedStation ? [selectedStation] : sortedStations;
    },
    googleMapLink() {
      var selectionsMade = this.selectedUserStation !== "" && this.selectedDestStation

      var station1 = this.orderedUserStations.find(station => {
          return station.number === this.selectedUserStation;
          }),
          station2 = this.orderedDestStations.find(station => {
          return station.number === this.selectedDestStation - 200;
          });

      return selectionsMade && this.positionsFilled ? this.generateMapLink(station1.position, station2.position) : ""
    }
  },
  data() {
    return {
      stations: "",
      selectedUserStation: null,
      selectedDestStation: null,
      user: {
        position: null,
        name: null,
        formatted_address: null
      },
      dest: {
        position: null,
        name: null,
        formatted_address: null
      },
      kmPerMin: 0.084,
      URL: "Pick a station from each column.",
      walkingDuration: "",
      stationsToDisplayUser: 5,
      stationsToDisplayDest: 5
    };
  },
  mounted() {
    axios
        .get(
          "https://api.jcdecaux.com/vls/v1/stations?contract=dublin&apiKey=f07f12786ba132c30596108298be8624fca5a48a"
        )
        .then(response => {
          this.stations = response.data;
          //this.updateDistToStation();
        })
        .catch(error => {
          console.log(error);
        });
  }
};
</script>

<style scoped>
#app {
  // Colours
  // Primary Normal: #004d40;
  // Primary Dark: #00251a;
  // Primary Light: #39796b;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  //text-align: center;
  margin: 0 auto;
  padding: 0;
  max-width: 480px;
}

div {
  margin: 0;
  padding: 0;
}

#inputs {
  width: 100%;
  margin: 0;
  padding: 16px;
  
}

.shadow {
  box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
}

.filled {
  color: white;
  background-color: #004d40;
  //margin: -16px 0 0 -8px;
  padding: 16px 16px 0 16px;
  //width: 100%
}

h3 {
  margin: 0;
  padding: 0 0 16px 0;
}

h2 {
  font-size: 1.4em;
  color: #00251a;
  margin: 0;
  padding: 16px 16px 0 16px;
  text-align: left;
}

p {
  font-style: italic;
  margin: 0;
  padding: 0;
}

.stations {
  margin: 0;
  padding: 0;
  display: block;
}

a {
  color: white;
  font-size: 2em
}

#link {
  display:block;
  box-sizing: border-box;
  background-color: #39796b;
  color: #00251a;
  text-align: center;
  padding: 16px
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

</style>
