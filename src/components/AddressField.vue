<template>
  <div>
    <input ref="autocomplete" 
        placeholder="Search" 
        class="search-location"
        onfocus="value = ''" 
        type="text" />
        <br/>
        <p v-if="place">{{place.formatted_address}}</p>
        <p v-if="place">{{place.formatted_address.location}}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Selected place
      place: null,
      // The 'Autocomplete' object from Google
      autocomplete: null
    };
  },
  mounted() {
    this.autocomplete = new google.maps.places.Autocomplete(
      // this.$refs is just the Vue version of 'getElementById'
      this.$refs.autocomplete,
      { types: ["geocode"] }
    );
    console.log(this.autocomplete);
    this.autocomplete.addListener("place_changed", () => {
      this.place = this.autocomplete.getPlace();

      // The data will be much easier to read in the console:
      console.log(this.place);
      // let ac = place.address_components;
      // let lat = place.geometry.location.lat();
      // let lon = place.geometry.location.lng();
      // let city = ac[0]["short_name"];
    });
  }
};
</script>

<style>
</style>