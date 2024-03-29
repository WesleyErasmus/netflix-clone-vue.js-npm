<template>
  <div class="search-page">
    <!-- NAVBAR COMPONENT -->
    <TopNavBarComponent />
    <!-- NAVBAR COMPONENT -->

    <!-- Header back btn -->
    <div class="back-btn-and-header-container">
      <router-link :to="{ path: '/home' }">
        <i class="fa-solid fa-arrow-left"></i>
      </router-link>
      <h1 class="search-page-header">Search for Movies In your Watch List</h1>
    </div>

    <!-- Searchbar container -->
    <div class="search-container">
      <!-- Searchbar form -->
      <form class="searchbar-form" action="/search" method="get">
        <!-- Searchbar input -->
        <input
          v-model="search"
          class="search"
          type="search"
          name="q"
          placeholder="Search..."
        />
        <!-- Searchbar icon -->
        <label class="button searchbutton" for="searchright"
          ><span class="mglass">&#9906;</span></label
        >
      </form>
    </div>
    <!-- Clear watch list button -->
    <button @click="clearWatchList" class="clear-watch-list-btn">
      Clear watch list
    </button>

    <!-- Watch list section wrapper -->
    <section id="watchList" class="movie-slider-container row">
      <!-- Movie slider nesting all "looped-over" slider items -->
      <!-- Slider card items -->
      <div class="card" v-for="(movie, id) in filteredMovies" :key="id">
        <!-- Looping though link using movie id(index) to render a unique images for each movie instance -->
        <img
          class="card-img-top"
          :src="'https://picsum.photos/200/300?' + movie.id"
        />

        <!-- Adding coming soon banner conditionally to all "coming soon movies"  -->
        <div
          v-if="movie.is_coming_soon == 1"
          :class="{
            comingSoonThumbnailBanner: movie.is_coming_soon == 1,
          }"
        >
          COMING SOON
        </div>

        <!-- Watch-list card body -->
        <!-- Movie card body -->
        <div class="card-body">
          <button class="card-show-more-btn" v-on:click="movie.SH = !movie.SH">
            <i class="fa-solid fa-chevron-down"></i>
          </button>
          <div class="card-inner-body">
            <!-- Movie card title -->
            <div class="card-title">{{ movie.name }}</div>

            <div v-show="movie.SH" class="v-show-start">
              <!-- Btn to add movies to the watch-list -->
              <button
                data-bs-toggle="tooltip"
                data-bs-placement="top"
                title="Remove from your watch list"
                class="watch-list-add-btn"
                v-on:click="removeMovieFromWatchList(movie)"
              >
                <i class="fa-solid fa-minus"></i>
              </button>
              <!-- Movie duration and release date -->
              <div class="card-duration-and-release-date">
                <div>{{ movie.duration }}m</div>
                <div>{{ movie.release_date }}</div>
              </div>

              <div class="rating">Rating: {{ movie.rating }} / 10</div>
              <!-- Movie description -->
              <div class="card-description">{{ movie.description }}</div>
              <!-- Movie actors -->
              <h6 class="actor-heading">Cast:</h6>
              <!-- V-for to loop through the actors nested array -->
              <div v-for="actor in movie.actors" :key="actor.id">
                <!-- Actor names -->
                <div class="actor-name">
                  {{ actor.first_name }} {{ actor.last_name }}
                </div>
              </div>
            </div>
            <!-- End of v-show -->
          </div>
          <!-- end of inner body -->
        </div>
      </div>
      <!-- End of movie card -->
      <!-- End of movie slider container -->
    </section>
    <!-- End of watch-list section -->

    <!-- Toast component for messages of duplicate movies and max movies in watch list -->
    <ToastComponent />

    <!-- FOOTER COMPONENT -->
    <FooterComponent />
    <!-- FOOTER COMPONENT -->
  </div>
</template>

<script>
// CodeSpace Movies API import
import axios from "axios";

// Top Navbar component import
import TopNavBarComponent from "../components/TopNavBarComponent.vue";

// // Footer component import
import FooterComponent from "../components/FooterComponent.vue";

// Toast import
import ToastComponent from "../components/ToastComponent.vue";

export default {
  components: {
    TopNavBarComponent,
    FooterComponent,
    ToastComponent,
  },
  data() {
    return {
      watchList: [],
      movies: [],
      search: "",
    };
  },
  computed: {
    // Search function to filter movies in the watch list
    filteredMovies() {
      return this.watchList.filter((movie) =>
        this.watchList.length
          ? Object.keys(this.watchList[0]).some((key) =>
              ("" + movie[key]).toLowerCase().includes(this.search)
            )
          : true
      );
    },
  },
  methods: {
    // Duplicate movie in watch list toast
    duplicateMovieAlert() {
      var x = document.getElementById("snackbar");
      x.className = "show";
      setTimeout(function () {
        x.className = x.className.replace("show", "");
      }, 3000);
    },

    // Watch list movie limit toast
    watchListFullAlert() {
      var x = document.getElementById("snackbar2");
      x.className = "show";
      setTimeout(function () {
        x.className = x.className.replace("show", "");
      }, 3000);
    },

    // Watch list movie successfully add toast
    watchListAddSuccessAlert() {
      var x = document.getElementById("snackbar3");
      x.className = "show";
      setTimeout(function () {
        x.className = x.className.replace("show", "");
      }, 3000);
    },

    // Add movies to watch-list function
    addMovieToWatchList(movie) {
      // Ensure that the watch-list array is not empty
      if (!this.watchList) {
        return;
      }

      // Prevents more than 20 movies from being added to the watch-list array
      if (this.watchList.length >= 20) {
        this.watchListFullAlert();
        return;
      }

      // Prevents duplicate movies in the watchList
      if (this.watchList.some((e) => e == movie)) {
        this.duplicateMovieAlert();
        return;
      }

      // Prevents coming soon movies in the watchList
      if (movie.is_coming_soon == 1) {
        return;
      }

      // Adding movies to the front of the watchList
      if (this.watchList.unshift(movie)) {
        // Movie added message
        this.watchListAddSuccessAlert();
      }

      // Save to local storage function
      this.saveMovieToLocalStorage();

      console.log(this.watchList);
    },

    // Remove movies from watch-list function
    removeMovieFromWatchList(movie) {
      const remove = this.watchList.filter((i) => i != movie);

      this.watchList = remove;

      this.saveMovieToLocalStorage();
    },

    // Save movies to watch-list function
    saveMovieToLocalStorage() {
      const parsed = JSON.stringify(this.watchList);
      localStorage.setItem("movies-in-watch-list", parsed);
    },

    // Clear watch list
    clearWatchList () {
      if (localStorage.length > 0) {
        localStorage.removeItem("movies-in-watch-list")
      }
      window.location.reload()
    },
  },
  mounted() {
    // FInally display local storage movies in the dom
    if (localStorage.getItem("movies-in-watch-list")) {
      try {
        this.watchList = JSON.parse(
          localStorage.getItem("movies-in-watch-list")
        );
      } catch (e) {
        localStorage.removeItem("movies-in-watch-list");
      }
    }

    // CodeSpace API import using axios
    axios
      .get("https://project-apis.codespace.co.za/api/movies")
      .then((response) => {
        this.movies = response.data.data;
        console.warn(response);
      });
  },
};
</script>

<style lang="scss">
// STYLING AND MEDIA QUERIES for the back button are on FilteredMovies.vue

// Clear watch-list button
.clear-watch-list-btn {
  z-index: 20;
  float: right;
  position: relative;
  bottom: 0vw;
  font-size: calc(7px + 0.5vw);
  background: transparent;
  background: rgba(0, 0, 0, 0.793);
  padding: 0.7rem 1rem;
  border: #141414 1px solid;
  margin-right: 4%;
  border-radius: 5px;
  color: rgb(255, 255, 255);
  text-shadow: rgba(0, 0, 0, 0.15) 1.95px 1.95px 2.6px;
}

.clear-watch-list-btn:hover {
  color: var(--primary-color);
}
</style>
