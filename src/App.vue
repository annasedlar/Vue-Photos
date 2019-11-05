<template>
  <div id="app">
    <div>
      <h1>Vue Photo Gallery</h1>
      <h6>Search for Photos: </h6>
      <form class="form-group fg--search" @submit.prevent="searchUnsplash(searchKeyword, 20)">
        <input type="text" class="input" placeholder="search" v-model="searchKeyword">
        <button type="submit"><font-awesome-icon icon="search" /></button>
    </form>
    </div>
    <div class="container">
      <div class="button-wrapper">
        <button class="btn" @click="searchUnsplash('Autumn', 20)">Autumn</button>

        <button class="btn" @click="searchUnsplash('Winter', 20)">Winter</button>

        <button class="btn" @click="searchUnsplash('Spring', 20)">Spring</button>

        <button class="btn" @click="searchUnsplash('Summer', 20)">Summer</button>
      </div>
      <stack
              :column-min-width="300"
              :gutter-width="15"
              :gutter-height="15"
              monitor-images-loaded
      >
        <stack-item
                v-for="(image, i) in images"
                :key="i"
                style="transition: transform 300ms"
        >
          <img :src="image.urls.small" :alt="image.alt_description" />
        </stack-item>
      </stack>
    </div>
    <button v-if="showLoadMoreButton" @click="loadMore" class="load-more-btn">Load More</button>
  </div>
</template>

<script>
import axios from "axios";
import { Stack, StackItem } from "vue-stack-grid";

export default {
  name: "app",
  components: {
    Stack,
    StackItem
  },
  created () {
    // fetch the inital images
    this.searchUnsplash('stars', 20);
  },

  data: () => ({
    images: [],
    searchKeyword: '',
    showLoadMoreButton: false,
    previousSearchWord: ''
  }),

  methods: {
    searchUnsplash(topic, number) {
      this.searchKeyword = topic;
      this.images = [];
      axios
        .get(
          `https://api.unsplash.com/search/photos?query=${this.searchKeyword}&per_page=${number}`,
          {
            headers: {
              Authorization:
                `Client-ID ${process.env.VUE_APP_UNSPLASHED_KEY}`,
              "Accept-Version": "v1"
            }
          }
        )
        .then(response => {
          this.images = response.data.results;
          this.showLoadMoreButton = true;
          this.previousSearchWord=this.searchKeyword;
          this.searchKeyword='';
        })
        .catch(() => {
          this.images = [];
          this.showLoadMoreButton = false;
        });
    },
    
    loadMore() {
      let imageCount = this.images.length;
      console.log(this.searchKeyword)
      console.log(imageCount + 10)
      axios
        .get(
          `https://api.unsplash.com/search/photos?query=${this.searchKeyword}&per_page=${imageCount + 10}`,
          {
            headers: {
              Authorization:
                `Client-ID ${process.env.VUE_APP_UNSPLASHED_KEY}`,
              "Accept-Version": "v1"
            }
          }
        )
        .then(response => {
          console.log(response)
          this.images.push(response.data.results);
          this.showLoadMoreButton = false;
        })
        .catch(e => {
          console.log(e)
          this.images = [];
          this.showLoadMoreButton = false;
        });
    }

  }
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

img {
  width: 100%;
  height: auto;
  border-radius: 12px;
}

.container {
  width: 80vw;
  margin: 0 auto;
}
.button-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 25px;
}

.btn {
  font-size: 18px;
  background-color: #42b983;
  color: white;
  padding: 10px 20px;
}

.load-more-btn {
  font-size: 18px;
  background-color: lightblue;
  color: white;
  padding: 10px 20px;
}

.btn:not(:last-child) {
  margin-right: 10px;
}


h6 {
  display: inline;
}

.fg--search input {
  display: inline;
  width: 25%;
  padding: 6px;
  margin: 10px;
}

.fg--search button {
  border: none;
  cursor: pointer;
  font-size: 16px;
  z-index: 2;
}

.fg--search input:focus + button .fa-search {
  color: darkgrey;
}

</style>
