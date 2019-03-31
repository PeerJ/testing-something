<template>
    <div id="app">
        <v-app id="inspire">
            <v-navigation-drawer
                    fixed right
                    clipped
                    class="grey lighten-4"
                    app temporary
                    v-model="drawer"
            >
                <v-list
                        dense
                        class="grey lighten-4"
                >
                    <template v-for="(item, i) in items">
                        <v-layout
                                row
                                v-if="item.heading"
                                align-center
                                :key="i"
                        >
                            <v-flex xs6>
                                <v-subheader v-if="item.heading">
                                    {{ item.heading }}
                                </v-subheader>
                            </v-flex>
                        </v-layout>
                        <v-divider
                                dark
                                v-else-if="item.divider"
                                class="my-3"
                                :key="i"
                        ></v-divider>
                        <v-list-tile
                                :key="i"
                                v-else
                                :href="item.href"
                        >
                            <v-list-tile-action>
                                <v-icon>{{ item.icon }}</v-icon>
                            </v-list-tile-action>
                            <v-list-tile-content>
                                <v-list-tile-title class="grey--text">
                                    {{ item.text }}
                                </v-list-tile-title>
                            </v-list-tile-content>
                        </v-list-tile>
                    </template>
                </v-list>
            </v-navigation-drawer>
            <v-toolbar color="primary lighten-1" app absolute clipped-left>
                <div class="title peerj-title" style="letter-spacing: -0.02em !important;">PeerJ<span class="text">-SciHub</span></div>

                <v-flex xs10 sm9 md7 ml-4 mt-2 peerj-search-bar>
                    <v-form @submit.prevent="sendSearch">
                        <v-text-field
                                v-model="searchText"
                                @keyup="sendSearch"
                                :error-messages="searchError"
                                ref="search"
                                solo-inverted
                                flat color="light-blue lighten-4"
                                label="Search for peer-reviewed PDFs at PeerJ."
                                append-icon="search"
                                clearable
                        ></v-text-field>
                    </v-form>
                </v-flex>

            <v-spacer class="hidden-sm-and-down"></v-spacer>
            <v-btn class="hidden-sm-and-down primary lighten-2" color="grey lighten-2" flat @click="sendSearch(...arguments, true)">Feeling Stochastic?!</v-btn>
            <v-toolbar-side-icon @click.native="drawer = !drawer"></v-toolbar-side-icon>
            </v-toolbar>

            <v-content>
                <v-container fluid fill-height class="lighten-4">
                    <v-layout row>
                        <v-flex md9 offset-md1 class="results-container">
                            <div class="row">
                                <div v-if="searchText" class="result-count">{{ totalResults.toLocaleString() }} result<span v-if="totalResults != 1">s</span></div>
                            </div>

                            <!--<div class="row" v-if="searchError">-->
                            <!--<div class="col-8-sm">-->
                            <!--<span class="alert alert-danger">{{ searchError }}</span>-->
                            <!--</div>-->
                            <!--</div>-->
                            <div class="result-list">
                                <div>
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
                                            <v-tooltip slot="append" left color="pink lighten-1">
                                                <a slot="activator"
                                                    :href="'https://peerj.com/articles/' + result._source.key + '.pdf'" class="result-pdf-link">
                                                    <v-btn fab dark small color="pink">
                                                        <v-icon dark>save_alt</v-icon>
                                                    </v-btn>
                                                </a>
                                                <span>Download PDF</span>
                                            </v-tooltip>
                                            <span class="result-citations" v-if="result._source.maxcitations > 0">{{ result._source.maxcitations }} citation<span v-if="result._source.maxcitations > 1">s</span></span>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div v-if="pagination.pages > 1 && !isHidden">
                                <v-pagination
                                        v-model="pagination.resultPage"
                                        @input="sendQuery"
                                        :length="pagination.pages"
                                        :total-visible="8"
                                ></v-pagination>
                            </div>
                        </v-flex>
                    </v-layout>
                </v-container>

                <v-footer
                        height="auto"
                            color="inverted lighten-1"
                        style="font-size: 16px"
                >
                    <v-layout
                            justify-center
                            row
                            wrap
                            py-3
                    >
                        <v-flex
                                inverted
                                lighten-1
                                py-3
                                dark--text
                                text-xs-center
                                xs5
                        >
                                We've hacked PeerJ, so you don't have to ;).
                                <div><a href="https://peerj.com/blog/?p=115284881385&preview=true">Read the April 1st announcement</a></div>
                        </v-flex>
                            <v-flex inverted
                                    lighten-1
                                    py-3
                                    px-3
                                    text-xs-center
                                    dark--text
                                    xs7>
                                "By 'Sci-hubbing' itself, PeerJ has taken Open Access to an 11" – Anon
                            </v-flex>
                    </v-layout>
                </v-footer>
            </v-content>
        </v-app>
    </div>
</template>

<script>
// import 'bootstrap/dist/css/bootstrap.min.css'
import moment from 'moment'

// JQuery is required by bootstrap, but let's try out a different ajax lib
import Axios from 'axios'
const itemsPerPage = 20;

export default {
  name: 'app',
    data () {
         return {
             currentQuery: '',
             totalResults: 0,
             pageCurrent: 0,
             page: 0,
             pagination: {
                 resultPage: 1,
                 pages: 0
             },
             searchError: '',
             searchText: '',
             searchResults: [],
             isHidden: false,
             drawer: null,
             items: [
                 { heading: 'Journals' },
                 { icon: 'home', href: 'https://peerj.com/', text: 'PeerJ – Life, Environ., Med' },
                 { icon: 'home', href: 'https://peerj.com/computer-science', text: 'PeerJ Computer Science' },
                 { icon: 'home', href: 'https://peerj.com/chemistry/', text: 'PeerJ Chemistry' },
                 { icon: 'home',  href: 'https://peerj.com/preprints/', text: 'PeerJ Preprints' },
                 { divider: true }
             ]
         }
     },
    filters: {
        formatDate: function (value) {
            if (!value) return '';
            return moment(String(value)).format('D MMM YYYY');
        }
    },
     methods: {
      scrollToTop(scrollDuration) {
          var scrollStep = -window.scrollY / (scrollDuration / 15),
              scrollInterval = setInterval(function(){
                  if ( window.scrollY != 0 ) {
                      window.scrollBy( 0, scrollStep );
                  }
                  else clearInterval(scrollInterval);
              },15);
      },
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

        if (this.timer) {
          clearTimeout(this.timer);
          this.timer = null;
        }
        this.timer = setTimeout(() => {
            this.sendQuery()
        }, 100);
     },
     sendQuery (resultPage) {
         this.searchError = ''
         let query = 'https://peerj.com/api/search/?' + this.currentQuery
         if (resultPage) {
             this.page = this.pagination.resultPage - 1
         }
         query += '&from=' + (this.page * itemsPerPage)
         Axios.get(query, {crossDomain: true})
              .then((result) => {
                  if (result.data.hits != 'undefined') {
                      if (result.data.hits.total) {
                          this.totalResults = result.data.hits.total
                          this.searchResults = result.data.hits.hits
                          this.pagination.resultPage = this.page + 1
                          this.pagination.pages = Math.ceil(result.data.hits.total / itemsPerPage)
                      } else {
                          this.searchError = 'No results found'
                      }
                  }

                  this.scrollToTop(200)
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
    #keep main .container {
        height: 660px;
    }

    .navigation-drawer__border {
        display: none;
    }

    .text {
        font-weight: 400;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif !important;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
        font-size: x-large;
    }

    .peerj-search-bar {
        margin-right: 15px;
    }

    .peerj-title {
        width: 127px !important;
        color: #204372;
    }

    @media (min-width: 860px) {
        .results-container {
            margin-left: 152px !important;
        }
    }

    .search-highlight {
        font-weight: 600;
        font-size: 105%;
        letter-spacing: -0.01em;
    }

    .result-list {
        max-width: 900px;
        font-family: 'Avenir', Helvetica, Arial, sans-serif !important;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
        font-size: x-large;
    }

    .result-item {
        margin-bottom: 12px; display: inline-block;
    }

    .result-left-item {
        display: inline-block;
        float: left;
        max-width: 78%;
        line-height: 18px;
    }

    .result-right-item {
        width: 18%;
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
        font-size: 20px;
        line-height: 1.2;
        margin-bottom: 8px;
        text-decoration: none;
    }

    .result-title:hover {
        text-decoration: underline;
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

    .result-pdf-link_old {
        color: deeppink;
        font-weight: bold;
        text-decoration: underline;
        padding: 3px 2px;
        border-radius: 2px;
    }

    .result-pdf-link {
        text-decoration: none;
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
