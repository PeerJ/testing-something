<template>
  <div id="app">
          <div class="col-12-sm">
              <input type="text" v-model="searchText" @keyup.enter="sendSearch">
              <button type="button" class="btn btn-primary" @click="sendSearch()">Search</button>
              <button type="button" class="btn btn-success" @click="sendSearch(...arguments, true)">Surprise me!</button>
              {{ totalResults }}
          </div>
          <div class="col-12-sm">
              <span v-if="searchError" class="alert alert-danger">{{ searchError }}</span>
          </div>
          <div class="col-12-sm">
              <ol :start="currentStart">
                  <li v-for="result in searchResults">{{ result._source.title }}</li>
              </ol>
          </div>
          <div class="col-12-sm">
              <nav v-if="searchResults.length">
                  <ul class="pagination">
                      <li class="page-item">
                          <a class="page-link" href="#" tabindex="-1" @click="sendQuery(page - 1)">Previous</a>
                      </li>
                      <li class="page-item">
                          <a class="page-link" href="#" @click="sendQuery(0)">1</a></li>
                      <li class="page-item active">
                          <a class="page-link" href="#" @click="sendQuery(1)">2 <span class="sr-only">(current)</span></a>
                      </li>
                      <li class="page-item">
                          <a class="page-link" href="#" @click="sendQuery(2)">3</a>
                      </li>
                      <li class="page-item">
                          <a class="page-link" href="#" @click="sendQuery(page + 1)">Next</a>
                      </li>
                  </ul>
              </nav>
          </div>
  </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.min.css'
// JQuery is required by bootstrap, but let's try out a different ajax lib
import Axios from 'axios'
const itemsPerPage = 25

export default {
  name: 'app',
     data () {
         return {
             currentQuery: '',
             pageCurrent: 0,
             page: 0,
             searchError: '',
             searchText: 'exemplo',
             searchResults: [],
             totalResults: 0,
         }
     },
     methods: {
         sendSearch (e, random) {
             if (!random && this.searchText.length < 3) {
                 this.searchError = 'Please type more than two letters'
                 return
             }
             this.page = 0

             // needs escaping
             this.currentQuery = random ? 'stochastic=1' : 'q=' + this.searchText
             this.sendQuery()
         },
         sendQuery (page) {
             let query = 'http://localhost:1180/api/search/?' + this.currentQuery
             if (page) {
                 this.page = page
             }
             query += '&from=' + this.page
             Axios.get(query, {crossDomain: true})
                  .then((result) => {
                      if (result.data.hits != 'undefined') {
                          if (result.data.hits.total) {
                              this.totalResults = result.data.hits.total
                              this.searchResults = result.data.hits.hits
                          } else {
                              this.searchError = 'No results found'
                          }
                      }
                  })
                  .catch((error) => {
                      console.log(error)
                  })
         }
     },
     computed: {
         currentStart() {
             return (this.page * itemsPerPage) + 1
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
    font-size: x-large;
}
</style>
