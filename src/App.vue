<template>
  <AddressField/>
</template>

<script>
import axios from "axios";
import AddressField from "./components/AddressField"

export default {
  //  Dublin Bikes API Key
  //  f07f12786ba132c30596108298be8624fca5a48a
  name: "App",
  components: {
    AddressField
  },
  
  data () {
    return {
      dbinfo: {}
    }
  }/*, // This works, I just have it turned off.
  mounted () {
    axios.get('https://api.jcdecaux.com/vls/v1/stations?contract=dublin&apiKey=f07f12786ba132c30596108298be8624fca5a48a')
      .then((response) => {
        console.log(response.data);
        this.dbinfo = response.data;
      })
      .catch((error) => {
        console.log(error);
      });
  }*/
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
