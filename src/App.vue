<template>
  <div id="app">
          <div class="row">
              <form class="form-inline" style="width: 100%" @submit.prevent="sendSearch()">
                  <div class="form-group bmd-form-group" style="width: 100%">
                      <label for="search" class="bmd-label-floating">Search PDFs at PeerJ. We've hacked ourselves, so you don't have to ;)</label>
                      <input ref="search" id="search" type="text" class="form-control col-8" name="search" aria-label="Search..." v-model="searchText" @keyup.enter="sendSearch">
                      <div class="input-group-append col-4">
                          <button class="btn btn-outline-info active" type="button" @click="sendSearch()">Search</button>
                          <button class="btn btn-sm btn-dark" type="button" @click="sendSearch(...arguments, true)">Feeling stochastic!</button>
                      </div>
                  </div>
              </form>
          </div>
          <div class="row">
              <div class="col-12-sm result-count">{{ totalResults }}</div>
          </div>

          <div class="row" v-if="searchError">
              <div class="col-8-sm">
                  <span class="alert alert-danger">{{ searchError }}</span>
              </div>
          </div>
            <div class="row">
              <div class="col-8-sm result-list">
                  <div :start="currentStart">
                      <div v-for="result in searchResults" class="result-item">
                          <div class="result-left-item">
                              <a :href="'https://peerj.com/articles/' + result._source.key + '/'" class="result-title" v-html="result.highlight.title[0]"></a>

                              <div>
                                  <span class="result-published">{{ result._source.published | formatDate }} - </span>
                                  <span v-show="isHidden"  class="result-fulltext">{{ result._source.abstract }}</span>
                                  <span v-show='!isHidden' class="result-fulltext" v-html="result.highlight.fulltext[0]"></span>
                              </div>
                          </div>
                          <div class="result-right-item">
                              <a
                                  :href="'https://peerj.com/articles/' + result._source.key + '.pdf'" class="result-pdf-link">[PDF]</a>
                              <span class="result-citations" v-if="result._source.maxcitations > 0">{{ result._source.maxcitations }} citation<span v-if="result._source.maxcitations > 1">s</span></span>
                          </div>
                      </div>
                  </div>
              </div>
          </div>
          <div class="col-12-sm" v-show='!isHidden'>
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
// import 'bootstrap/dist/css/bootstrap.min.css'
import moment from 'moment'

// JQuery is required by bootstrap, but let's try out a different ajax lib
import Axios from 'axios'
const itemsPerPage = 25;

export default {
  name: 'app',
     data () {
         return {
             currentQuery: '',
             pageCurrent: 0,
             page: 0,
             searchError: '',
             searchText: '',
             searchResults: [],
             totalResults: '',
             isHidden: false
         }
     },
    filters: {
        formatDate: function (value) {
            if (!value) return '';
            return moment(String(value)).format('D MMM YYYY');
        }
    },
     methods: {
         sendSearch (e, random) {
             if (!random && this.searchText.length < 3) {
                 this.searchError = 'Please type more than two letters'
                 this.$refs.search.focus()
                 return
             }

             if (random) {
                 this.searchText = ''
                 this.isHidden = true
             } else {
                 this.isHidden = false
             }

             this.page = 0

             this.currentQuery = random ? 'stochastic=1' : 'q=' + encodeURIComponent(this.searchText)
             this.sendQuery()
         },
         sendQuery (page) {
             this.searchError = ''
             let query = 'https://peerj.com/api/search/?' + this.currentQuery
             if (page) {
                 this.page = page
             }
             query += '&from=' + this.page
             Axios.get(query, {crossDomain: true})
                  .then((result) => {
                      if (result.data.hits != 'undefined') {
                          if (result.data.hits.total) {
                              this.totalResults = result.data.hits.total + ' result(s)'
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
        color: #2c3e50;
        margin-top: 60px;
        font-size: x-large;
    }

    .search-highlight {
        font-weight: 600;
        font-size: 105%;
        letter-spacing: -0.01em;
    }

    .result-list {
        max-width: 900px;
    }

    .result-item {
        margin-bottom: 12px; display: inline-block;
    }

    .result-left-item {
        display: inline-block;
        float: left;
        max-width: 83%;
        line-height: 18px;
    }

    .result-right-item {
        width: 15%;
        float:right;
        display: inline-block;
        margin-left: 10px;
        font-size: 12px;
        font-weight: bold;
        line-height: 20px;
    }

    .result-title {
        color: #0c37eb !important;
        font-weight: 500;
        font-size: 18px;
        margin-bottom: 8px;
    }

    .result-published {
        font-size: 13px;
        line-height: 1.2;
        color: #717171;
        font-weight: 500;
    }

    .result-fulltext {
        font-size: 14px;
        line-height: 1.2;
        font-weight: 500;
        color: #515151;
    }

    .result-pdf-link {
        color: deeppink; font-weight: bold
    }

    .result-citations {
        margin-left: 10px;
        font-size: 14px;
        color: green;
        font-weight: 400
    }
    .result-count {
        font-weight: bold;
        font-size: 12px;
        margin-bottom: 15px;
    }
</style>
