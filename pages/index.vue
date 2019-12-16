<template>
  <div>
    <section id="affCarousel" class="carousel slide mb-5" data-ride="carousel">
      <b-carousel
        id="carousel-1"
        :interval="4000"
        controls
        indicators
        background="#000"
      >
        <!-- Slides with image only -->
        <b-carousel-slide class="carousel-item" img-src="@/assets/images/slide-3.jpg" />
        <!-- Slides with image only -->
        <b-carousel-slide img-src="@/assets/images/slide-2.jpg" />
        <!-- Slides with image only -->
        <b-carousel-slide img-src="@/assets/images/slide-4.jpg" />
      </b-carousel>
      <Countdown />
    </section>

    <main class="container">
      <section class="row filter-row">
        <template v-for="(btn, index) in filterButtons">
          <div :key="index" class="col-4">
            <button :id="btn.name + '-btn'" :class="btn.isActive ? 'btn btn-secondary active' : 'btn btn-secondary'" :disabled="disableButtons" @click="filterPosts(btn.type)">
              <template v-if="btn.icon">
                <i :class="'fab ' + btn.icon" /> <span class="d-none d-md-inline">{{ btn.label }}</span>
              </template>
              <template v-else>
                {{ btn.label }}
              </template>
            </button>
          </div>
        </template>
      </section>
      <section class="grid-sizer">
        <template v-if="filteredPosts">
          <div id="post-list" v-masonry transition-duration="0.4s" item-selector=".grid-item" class="masonry-container grid row">
            <template v-for="post in filteredPosts">
              <article v-if="post.service_slug === 'twitter'" :key="post.id" v-masonry-tile class="col-12 col-md-6 col-lg-4 grid-item twitter-post">
                <div class="card card-block text-center">
                  <div class="post-type">
                    <i class="fab fa-twitter" />
                  </div>
                  <h4><a :href="'https://twitter.com/' + post.item_data.user.username" target="_blank">{{ post.item_data.user.username }}</a></h4>
                  <blockquote class="card-blockquote">
                    <!-- eslint-disable-next-line -->
                    <p v-html="parseTwitter(post.item_data.tweet)" />
                    <footer>
                      <small v-text="formatDate(post.item_published)" />
                    </footer>
                  </blockquote>
                </div>
              </article>
              <article v-if="post.service_slug === 'instagram'" :key="post.id" v-masonry-tile class="col-12 col-md-6 col-lg-4 grid-item instagram-post">
                <div class="card">
                  <div class="post-type">
                    <i class="fab fa-instagram" />
                  </div>
                  <a :href="post.item_data.link">
                    <div v-lazy-container="{ selector: 'img' }">
                      <img
                        class="card-img-top img-fluid"
                        :data-src="post.item_data.link + 'media/?size=l'"
                        :alt="post.item_name"
                        :data-loading="loadingIcon"
                      >
                    </div>
                  </a>
                  <div class="card-block">
                    <h4><a :href="'https://instagram.com/' + post.item_data.user.username" target="_blank">{{ post.item_data.user.username }}</a></h4>
                    <!-- eslint-disable-next-line -->
                    <p class="card-text" v-html="parseInstagram(post.item_data.caption)" />
                    <p class="card-text">
                      <small class="text-muted" v-text="formatDate(post.item_published)" />
                    </p>
                  </div>
                </div>
              </article>
              <article v-if="post.service_slug === 'manual'" :key="post.id" v-masonry-tile class="col-12 col-md-6 col-lg-4 grid-item manual-post">
                <div class="card">
                  <div class="post-type">
                    AFF
                  </div>
                  <a :href="post.item_data.link" target="_blank">
                    <!-- Images from http://aff-stream-bullring.383apps.co.uk/ not loading as in maintenance mode-->
                    <!-- <img class="card-img-top img-fluid" :src="post.item_data.image_url" alt="Card image cap"> -->
                    <!-- <img class="card-img-top img-fluid" src="https://placeimg.com/640/480/people" alt="Card image cap"> -->
                    <div v-lazy-container="{ selector: 'img' }">
                      <img
                        class="card-img-top img-fluid"
                        :data-src="'https://placeimg.com/640/480/people'"
                        :alt="post.item_name"
                        :data-loading="loadingIcon"
                      >
                    </div>
                  </a>
                  <div class="card-block">
                    <p class="card-text">
                      {{ post.item_data.text }}
                    </p>
                    <div class="card-link text-center">
                      <a :href="post.item_data.link" target="_blank">{{ post.item_data.link_text }}</a>
                    </div>
                    <p class="card-text">
                      <small class="text-muted text-center" v-text="formatDate(post.item_published)" />
                    </p>
                  </div>
                </div>
              </article>
            </template>
          </div>
        </template>
        <div class="row">
          <div class="col-12 text-center">
            <div class="loading">
              <i class="fa fa-sync fa-spin fa-3x fa-fw margin-bottom" />
              <span class="sr-only">Loading...</span>
            </div>
            <br>
            <button id="loadMoreButton" class="btn btn-primary" :disabled="disableButtons" @click="loadMorePosts">
              <template v-if="isLoading">
                <i class="fa fa-sync fa-spin" />
              </template>
              <template v-else>
                Load More
              </template>
            </button>
          </div>
        </div>
      </section>
      <hr>
      <footer class="row credit">
        <div class="col-6">
          <p>
            © Alex Ross 2020<br>
            <a href="https://rosso.codes">https://rosso.codes</a>
          </p>
        </div>
        <div class="col-6">
          <p class="text-right">
            <logo class="w-25" />
          </p>
        </div>
      </footer>
    </main> <!-- /container -->
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import Logo from '~/components/Logo.vue'
import Countdown from '~/components/Countdown.vue'
import LoadingIcon from '@/assets/images/loading.svg'

export default {
  components: {
    Logo,
    Countdown,
    //LoadingIcon
  },
  data () {
    return {
      posts: [],
      allPosts: [],
      page: 1,
      numPosts: 20,
      filter: null,
      isLoading: false,
      disableButtons: false,
      loadingIcon: LoadingIcon,
      filterButtons: [
        {
          name: 'aff',
          type: 'manual',
          icon: null,
          label: 'AFF',
          isActive: false
        },
        {
          name: 'twitter',
          type: 'twitter',
          icon: 'fa-twitter',
          label: 'Twitter',
          isActive: false
        },
        {
          name: 'instagram',
          type: 'instagram',
          icon: 'fa-instagram',
          label: 'Instagram',
          isActive: false
        }
      ]
    }
  },
  computed: {
    filteredPosts () {
      let data = this.allPosts
      const _this = this
      const filterType = this.filter
      if (filterType) {
        data = data.filter((item) => {
          return item.service_slug === filterType
        })
      }
      _this.enableButtons()
      return data
    }
  },
  mounted () {
    const _this = this
    this.fetchPosts(this.page, this.numPosts)
    // this.$redrawVueMasonry('postList')
    this.$Lazyload.$on('loaded', function () {
      setTimeout(() => {
        _this.$redrawVueMasonry()
      }, 500)
    })
  },
  methods: {
    enableButtons () {
      const _this = this
      setTimeout(() => {
        _this.disableButtons = false
        _this.$redrawVueMasonry()
      }, 500)
    },
    fetchPosts (page, numPosts) {
      axios
        .get(`https://private-cc77e-aff.apiary-mock.com/posts?page=${page}&numPosts=${numPosts}`)
        .then((response) => {
          console.log(response)
          response.data.items.sort(function (a, b) {
            const dateA = moment(a.item_published)
            const dateB = moment(b.item_published)
            return dateB - dateA
          })
          this.posts = response.data.items
          this.allPosts = [...this.allPosts, ...this.posts]
          const _this = this
          setTimeout(() => {
            _this.isLoading = false
            _this.enableButtons()
          }, 500)
        })
        .catch(error => error)
    },
    loadMorePosts () {
      this.disableButtons = true
      this.isLoading = true
      this.page++
      this.fetchPosts(this.page, this.numPosts)
      // this.$redrawVueMasonry('postList')
    },
    parseTwitter (text) {
      text = text.replace(/[A-Za-z]+:\/\/[A-Za-z0-9-_]+\.[A-Za-z0-9-_:%&~?/.=]+/g, function (url) {
        return '<a href="' + url + '" target="_blank">' + url + '</a>'
      })
      text = text.replace(/[@]+[A-Za-z0-9-_]+/g, function (u) {
        const username = u.replace('@', '')
        return '<a href="https://twitter.com/' + username + '" target="_blank">' + u + '</a>'
      })
      text = text.replace(/[#]+[A-Za-z0-9-_]+/g, function (t) {
        const tag = t.replace('#', '%23')
        return '<a href="https://twitter.com/search?q=' + tag + '" target="_blank">' + t + '</a>'
      })
      return text
    },
    parseInstagram (text) {
      text = text.replace(/[A-Za-z]+:\/\/[A-Za-z0-9-_]+\.[A-Za-z0-9-_:%&~?/.=]+/g, function (url) {
        return '<a href="' + url + '" target="_blank">' + url + '</a>'
      })
      text = text.replace(/[@]+[A-Za-z0-9-_]+/g, function (u) {
        const username = u.replace('@', '')
        return '<a href="https://instagram.com/' + username + '" target="_blank">' + u + '</a>'
      })
      text = text.replace(/[#]+[A-Za-z0-9-_]+/g, function (t) {
        const tag = t.replace('#', '')
        return '<a href="https://instagram.com/explore/tags/' + tag + '" target="_blank">' + t + '</a>'
      })
      return text
    },
    filterPosts (type) {
      this.disableButtons = true
      if (this.filter === type) {
        this.filter = null
        const fb = this.filterButtons.map(el => ({ ...el, isActive: false }))
        this.filterButtons = fb
      } else {
        const fb = this.filterButtons.map(el => ({ ...el, isActive: false }))
        const fbb = fb.map(el => (el.type === type ? { ...el, isActive: true } : el))
        this.filterButtons = fbb
        this.filter = type
      }
    },
    formatDate (date) {
      return moment(date).format('dddd D MMMM YYYY')
    }
  }
}
</script>
