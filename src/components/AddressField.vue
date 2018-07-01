<template>
  <div>
    <input ref="autocomplete" 
        :placeholder=fieldPlaceholder
        class="search-location"
        onfocus="value = ''" 
        type="text"
        ></input>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Selected place
      place: null,
      position: {},
      name: null,
      formatted_address: null,
      // The 'Autocomplete' object from Google
      autocomplete: null
    };
  },
  props: {
    fieldPlaceholder: String
  },
  mounted() {
    var dublinBounds = new google.maps.LatLngBounds(
      new google.maps.LatLng(53.412807, -6.020695),
      new google.maps.LatLng(53.230086, -6.388394));

    this.autocomplete = new google.maps.places.Autocomplete(
      // this.$refs is just the Vue version of 'getElementById'
      this.$refs.autocomplete,
      { bounds: dublinBounds, types: ["geocode"],componentRestrictions: {country: 'ie'} }
    );
    this.autocomplete.addListener("place_changed", () => {
      this.place = this.autocomplete.getPlace();
      // The data will be much easier to read in the console:
      // let ac = place.address_components;
      // console.log(this.place)
      this.position.lat = this.place.geometry.location.lat();
      this.position.lng = this.place.geometry.location.lng();
      this.formatted_address = this.place.formatted_address;
      this.name = this.place.name;

      this.$emit("update:position", this.position);
      this.$emit("update:name", this.name)
      this.$emit("update:formatted_address", this.formatted_address)
      this.$emit("updateDistToStation")
      // let city = ac[0]["short_name"];
    });
  }
};
</script>

<style>
div {
  margin: 0;
  padding: 0;
}
input {
  margin: 0 16px;
  padding: 16px 0 0 0;
  border-top: 0;
  border-left: 0;
  border-bottom: 2px solid #00251a;
  border-right: 0;
  border-radius: 0;
  display: block;
  font-size: 1.2em;
  box-sizing: border-box;
}
// Classes for styling the autocomplete object.
.pac-container {
  width: 100%;
  margin: 0;
  padding: 0;
}

.pac-icon {
  margin: 0;
  padding: 0;
  font-size: 0;
  display: none
}

.pac-item {
  margin: 0;
  padding: 0;
  font-size: 1em;
  color: black;
}

.pac-item-query {
  margin: 0;
  padding: 0 0.35em 0 0;
  font-size: 1em;
  color: Red;
}

.pac-item-matched {
  margin: 0;
  padding: 0;
  font-size: 1em;
  color: black;
}
</style>