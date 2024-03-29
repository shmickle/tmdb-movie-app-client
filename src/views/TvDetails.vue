<template>
  <!-- Loading done & tv exists -->
  <div v-if="tvExists && !loadingDetails">
    <v-container class="d-xs-block d-sm-none trailer-container" v-if="tvVideos"
      ><iframe
        width="100%"
        height="215"
        :src="`https://www.youtube.com/embed/${youtubeID}`"
        frameborder="0"
        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
      ></iframe>
    </v-container>
    <div class="details-container" :style="{ background: detailsBackground }">
      <div class="overlay"></div>
      <v-container>
        <GenreChips :media="tv" class="pt-2 d-xs-block d-sm-none" />
        <v-row class="details-row">
          <v-col cols="12" sm="3"
            ><v-img :src="tvPoster" v-if="tv.poster_path"
          /></v-col>
          <v-col cols="12" sm="9" style="z-index: 1">
            <GenreChips :media="tv" class="mt-2 mt-sm-0 d-none d-sm-block" />

            <h1 class="tv-title">
              {{ tv.name || tv.original_name }}
              <sup class="text-caption"
                >({{
                  tv.first_air_date ? tv.first_air_date.split('-')[0] : 'N/A'
                }})</sup
              >
            </h1>
            <p class="tv-overview">{{ tv.overview }}</p>
            <p class="producer">
              Created By:
              <span v-for="(creator, index) in tvCreators" :key="index"
                ><router-link :to="`/person/${creator.id}`">
                  {{ creator.name }}</router-link
                >{{ index == tvCreators.length - 1 ? '' : ', ' }}</span
              >
            </p>
            <div class="d-flex align-center">
              <v-progress-circular
                :color="tvRating"
                rotate="270"
                size="64"
                width="5"
                :value="tv.vote_average * 10"
                >{{ Math.floor(tv.vote_average * 10) }}%</v-progress-circular
              >
              <AddRemoveButton
                class="ml-2"
                v-if="loggedIn"
                :mediaInfo="tv"
                rounded="true"
              />
            </div>
          </v-col>
        </v-row>
      </v-container>
    </div>
    <!-- Cast Section -->
    <v-container class="py-4">
      <h2 class="tv-section-heading my-5 text-center">Cast</h2>
      <MediaCarouselCards :info="tvCast" v-if="tvCast.length > 0" />
      <h3 v-else class="text-center">Nothing found.</h3>
      <!-- Similar Tv Section -->
      <h2 class="tv-section-heading my-5 text-center">Similar Shows</h2>
      <MediaCarouselCards :info="similarTv" v-if="similarTv.length > 0" />
      <h3 v-else class="text-center">Nothing found.</h3>
    </v-container>
  </div>
  <!-- Loading done & tv not found -->
  <v-container
    fill-height
    class="justify-center"
    v-else-if="!loadingDetails && !tvExists"
  >
    <div class="align-center">
      <h1 class="text-uppercase">Show Not Found</h1>
    </div>
  </v-container>
  <!-- Loading -->
  <v-container fill-height class="justify-center" v-else>
    <div class="d-flex align-center flex-column">
      <h1 class="mb-2 text-uppercase text-center">Loading details...</h1>
      <BaseLoadingRoller />
    </div>
  </v-container>
</template>

<script>
import { mapGetters } from 'vuex'
import AddRemoveButton from '../components/AddRemoveButton'
import apiClient from '../services/apiCalls'
import BaseLoadingRoller from '../components/BaseLoadingRoller'
import GenreChips from '../components/GenreChips'
import MediaCarouselCards from '../components/MediaCarouselCards'

export default {
  name: 'TvDetails',
  components: {
    AddRemoveButton,
    BaseLoadingRoller,
    GenreChips,
    MediaCarouselCards
  },
  data() {
    return {
      tv: {},
      tvExists: false,
      tvCast: [],
      tvCreators: [],
      tvVideos: [],
      similarTv: [],
      youtubeID: '',
      vimeoID: '',
      loadingDetails: true
    }
  },
  computed: {
    ...mapGetters(['getUID', 'loggedIn']),
    tvPoster() {
      return `https://image.tmdb.org/t/p/w500/${this.tv.poster_path}`
    },
    tvRating() {
      if (this.tv.vote_average < 5) {
        return 'error'
      } else if (this.tv.vote_average < 7) {
        return 'warning'
      } else {
        return 'success'
      }
    },
    detailsBackground() {
      if (this.tv.backdrop_path) {
        return `url(https://image.tmdb.org/t/p/original/${this.tv.backdrop_path}) no-repeat center/cover`
      } else if (this.tv.poster_path) {
        return `url(https://image.tmdb.org/t/p/original/${this.tv.poster_path}) no-repeat center/cover`
      } else {
        return `#efefef`
      }
    }
  },
  watch: {
    $route(to, from) {
      this.loadingDetails = true
      this.getTvDetails(to.params.id)
    }
  },
  methods: {
    getTvDetails(tvID) {
      apiClient
        .getTvDetails(tvID)
        .then(async response => {
          if (response.status == 200) {
            this.tv = response.data
            this.tvCast = this.tv.credits.cast
            this.tvCreators = this.tv.created_by
            this.tvVideos = this.tv.videos.results
            this.similarTv = this.tv.similar.results
            this.tvExists = true

            if (this.tvVideos.length > 1) {
              this.tvVideos.map(video => {
                if (video.type == 'Trailer') {
                  this.youtubeID = video.key
                }
              })
            } else if (this.tvVideos.length == 1) {
              this.youtubeID = this.tvVideos[0].key
            } else {
              this.youtubeID = ''
            }
          } else {
            console.log('error getting tv details')
          }
        })
        .catch(error => {
          console.log(error)
          this.tvExists = false
        })
        .finally(() => {
          this.loadingDetails = false
          document.title = `${this.tv.name} | My Media Lists`
        })
    }
  },
  created() {
    this.getTvDetails(this.$route.params.id)
  }
}
</script>

<style lang="scss" scoped>
.details-container {
  background: #efefef;
  color: white;
  position: relative;

  @media screen and (min-width: 600px) {
    padding: 1rem 0;
  }

  .overlay {
    position: absolute;
    z-index: 0;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.85);
  }

  .details-row {
    flex-direction: column-reverse;

    @media screen and (min-width: 600px) {
      flex-direction: row;
    }
  }
}

.trailer-container {
  line-height: 0;
  padding: 0;
}

.tv-title,
.tv-section-heading {
  font-size: 2em;
}

.tv-overview {
  font-size: 0.875em;

  @media screen and (min-width: 600px) {
    font-size: 1em;
  }
}

.producer {
  font-size: 0.875rem;

  a {
    color: #cae9ff;

    &:hover {
      opacity: 0.7;
    }
  }
}
</style>
