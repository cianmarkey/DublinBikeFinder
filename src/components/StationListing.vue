<template>
  <div class="station-listing"
      @click="reportSelected()"
      :class="classObject"
      >
    <p v-if="selected === true && wantBike === true">First head to</p>
    <p v-if="selected === true && wantBike === false">Then on to</p>
    <h3>{{ name }}</h3>
    <div v-if="wantBike">
      <div class="bikes"><span v-if="available_bikes < warningLimit">Only </span>{{available_bikes}} bike<span v-if="available_bikes > 1">s</span> available</div>
      <div v-if="!selected" class="dist">{{distToUser}} min<span v-if="distToUser > 1">s</span> walk from {{userName}}</div>
    </div>
    <div class="meta" v-else>
      <div><span v-if="available_bike_stands < warningLimit">Only </span>{{available_bike_stands}} bike stand<span v-if="available_bike_stands > 1">s</span> free</div>
      <div  v-if="!selected" class="dist">{{distToDest}} min<span v-if="distToDest > 1">s</span> walk to {{destName}}</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      warningLimit: 4
    };
  },
  props: {
    id: Number,
    name: String,
    userName: String,
    destName: String,
    position: {
      type: Object,
      default: function() {
        return { message: "Station's position not found." };
      }
    },
    banking: Boolean,
    bonus: Boolean,
    staus: String,
    bike_stands: Number,
    available_bike_stands: Number,
    available_bikes: Number,
    last_update: Number,
    selected: Boolean,
    distToUser: Number,
    distToDest: Number,
    wantBike: Boolean
  },
  computed: {
    classObject: function() {
      return {
        selected: this.selected,
        warning:
          (!this.selected &&
            (this.wantBike && this.available_bikes < this.warningLimit)) ||
          (!this.wantBike && this.available_bike_stands < this.warningLimit)
      };
    }
  },
  methods: {
    reportSelected: function() {
      if (!this.selected) {
        this.$emit("report-selected", this.id);
        this.$emit("update:selected", true);
      } else {
        this.$emit("report-selected", null);
        this.$emit("update:selected", false);
      }
    }
  }
};
</script>

<style scoped>
div {
  margin: 0;
  padding: 0;
}

.station-listing {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin: 16px auto;
  padding: 16px;
  border: 0;
  display: block;
  box-sizing: border-box;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
  cursor: pointer;
}

.station-listing:hover {
  background-color: #f5f5f5;
  //font-family: "Avenir", Helvetica, Arial, sans-serif;
  //-webkit-font-smoothing: antialiased;
  //-moz-osx-font-smoothing: grayscale;
  //text-align: left;
  //color: #2c3e50;
  //margin: 16px auto;
  //padding: 16px;
  //border: 0;
  //display: block;
  //box-sizing: border-box;
  //box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
}

#meta {
  margin: 0;
  padding: 0;
}

h3 {
  padding: 0 0 16px 0;
  margin: 0;
}

p {
  font-style: italic;
  margin: 0;
  padding: 0;
}

.selected {
  color: white;
  background-color: #004d40;
  margin: 0;
  padding: 0 16px 16px 16px;
  display: block;
  cursor: default;
  //width: 100%
}

.selected:hover {
  background-color: #004d40;
}

.warning {
  background-color: #ffb300;
  color: #333;
}

.warning:hover {
  background-color: #ffe54c;
}

span {
  margin: 0;
  padding: 0;
}
</style>
