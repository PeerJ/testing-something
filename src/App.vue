<template>
  <div id="app">
      <input type="text" v-model="searchText" @keyup.enter="sendSearch">
      <button type="button" class="btn btn-primary" @click="sendSearch()">Search</button>
      <button type="button" class="btn btn-success" @click="sendSearch(...arguments, true)">Surprise me!</button>
      <ol>
          <li v-for="result in searchResults">{{ result }}</li>
      </ol>
  </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.min.css'
// JQuery is required by bootstrap, but let's try out a different ajax lib
import Axios from 'axios'

export default {
  name: 'app',
     data () {
         return {
             searchText: 'exemplo',
             searchResults: [],
         }
     },
     methods: {
         sendSearch (e, random) {
             let query = random ? 'stochastic' : 'q=' + this.searchText
             if (this.searchText.length > 3) {
                 Axios.get('http://localhost:1180/api/search/?' + query, {crossDomain: true})
                      .then((result) => {
                          if (result.data.hits != 'undefined') {
                              this.searchResults = result.data.hits
                              console.log(result)
                          }

                      })
                      .catch((error) => {
                          console.log(error)
                      })
             }
         }
     }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
