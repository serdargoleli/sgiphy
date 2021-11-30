<template>
  <div class="home-page">
    <div class="container pt-5 is-fluid">
      <div class="content-box">
        <div class="text-white"></div>
        <div class="search-box">
          <input type="text" class="cs-input" placeholder="Gif arayın..." @keypress.enter="searchGif()" v-model="query" />
          <button class="btn btn-orange" @click="searchGif()">Search</button>
        </div>
        <p class="trending-search mt-4" v-if="populerSearchText.length > 0">
          <span class="text-gray">Populer Search: </span>
          <nuxt-link :to="`?search=${splitSpace(search)}`" v-for="(search, index) in populerSearchText" :key="index + '' + search">
            {{ search }}<span v-if="index != populerSearchText.length - 1" class="mr-1">, </span>
          </nuxt-link>
        </p>

        <div class="page-title search-title mt-5" v-if="$route.query.hasOwnProperty('search')">
          <h1 class="search-title">{{ $route.query.search }}<span class="small-text"> için arama sonuçları</span></h1>
          <button class="btn-clear" @click="clearSearch"><i class="mdi mdi-close"></i>Aramayı Temizle</button>
        </div>
        <div class="page-title mt-5" v-else>
          <h1><b-icon icon="trending-up" class="mr-2" size="is-medium"></b-icon>TREND GİFLER</h1>
        </div>

        <div class="loader mt-5" v-if="isLoading"></div>
        <template v-else>
          <section class="pt-3" v-if="gifs.length > 0">
            <div v-for="(gif, index) in gifs" :key="gif.id + ' ' + index" class="img-box" @click=";(isModal = true), (selectedGif = gif)">
              <img :src="gif.images.fixed_height.url" />
              <div class="img-box-hover">
                <a href="javascript:;" @click=";(isModal = true), (selectedGif = gif)">
                  <zoom-icon />
                </a>
              </div>
            </div>
          </section>
          <b-message type="is-info mt-4" v-else>
            <template v-if="$route.query.hasOwnProperty('search')">
              <b>{{ $route.query.search }}</b> için arama sonucu bulunamadı.</template
            >
          </b-message>
        </template>
      </div>
      <div class="is-flex is-justify-content-center pb-5" v-if="gifs.length > 0 && !isLoading">
        <button class="btn-orange outline" @click="getTrendingGifs()">Daha Fazlasını Görüntüle</button>
      </div>
    </div>

    <button class="btn-orange btn-go-top" @click="scrollToTop" v-if="gifs.length > 0 && scrollToTopView">
      <b-icon icon="arrow-collapse-up"></b-icon>
    </button>
    <!-- MODAL BOX -->
    <modal :selectedGif="selectedGif" :isModal.sync="isModal" />

    <!-- INFINITY SCROLL COMPONENT  -->
  </div>
</template>

<script>
import Modal from '~/components/modal.vue'
import ZoomIcon from '~/components/zoom.vue'
export default {
  name: 'HomePage',
  scrollToTop: true,
  data() {
    return {
      page: 0,
      isLoading: true,
      query: '',
      gifs: [],
      isModal: false,
      limit: 40,
      selectedGif: null,
      populerSearchText: [],
      offset: 0,
      windowWidth: 0,
      windowHeight: 0,
      scrollToTopView: false,
    }
  },
  components: {
    Modal,
    ZoomIcon,
  },
  async created() {
    // sayfa yenilendiğinde veya yeniden oluşturulduğunda ofseti sıfırladık
    // önce trend verileri çekip daha sonra arama yaptıımızda offset değerinin sıfırlanmasını istedik
    // Sayfa ilk yüklendiği anda
    if (!this.$route.query.hasOwnProperty('search')) {
      if (this.$store.state.gifs.length > 0) {
        this.gifs = this.$store.state.gifs
        this.page = 1
        this.isLoading = false
      } else {
        await this.getTrendingGifs()
      }
    } else {
      this.query = this.$route.query.search
      await this.getSearchGifs()
    }
    if (this.$store.state.populerSearchText.length > 0) {
      this.populerSearchText = this.$store.state.populerSearchText
    } else {
      await this.getPopulerSearchText()
    }
  },
  computed: {
    apiKey() {
      return this.$store.state.apiKey
    },
  },
  // url'de bulunan query objesi içerisinde değişikliği yakalamak için
  watch: {
    async $route() {
      // routing yönlendirmesi yapıldığında offseti ve page değerlerini sıfırladık
      this.offset = 0
      this.page = 0
      // Eğer url'de search sorgusu yoksa state alanında tuttuğumuz gifs datasını getir
      if (!this.$route.query.hasOwnProperty('search')) {
        if (this.$store.state.gifs.length > 0) {
          this.isLoading = true
          this.gifs = this.$store.state.gifs
          this.isLoading = false
          // Eğer state alanında veri varsa sayfayı 1 yap
          this.page = 1
        } else {
          await this.getTrendingGifs()
        }
        this.query = ''
      } else {
        this.query = this.$route.query.search
        this.isLoading = true
        await this.getSearchGifs(this.query)
      }
    },
  },
  mounted() {
    this.$nextTick(function () {
      window.addEventListener('scroll', this.getWindowHeight)
      //Init
      this.getWindowHeight()
      this.scrollToTopView = this.windowHeight > 350 ? true : false
    })
  },
  methods: {
    splitSpace(text) {
      return text.replace(/ /g, '-')
    },
    clearSearch() {
      this.gifs = this.$store.state.gifs
      let query = Object.assign({}, this.$route.query)
      delete query.search
      this.$router.replace({ query })
      this.query = ''
    },
    scrollToTop() {
      window.scrollTo({ top: 0 })
    },
    searchGif() {
      this.$router.push({ query: { search: this.splitSpace(this.query) } })
    },
    async getTrendingGifs() {
      this.page += 1
      this.offset = this.page * this.limit - this.limit
      // Trend gifleri çağırmak için
      let url = `/gifs/trending?api_key=${this.apiKey}&limit=${this.limit}&offset=${this.offset}`
      if (this.$route.query.hasOwnProperty('search')) {
        let query = this.$route.query.search
        url = `/gifs/search?api_key=${this.apiKey}&q=${this.splitSpace(query)}&limit=${this.limit}&offset=${this.offset}`
      }
      await this.$axios.$get(url).then((res) => {
        if (res.meta.status === 200) {
          this.gifs = this.gifs.concat(res.data)
          if (this.page === 1 && !this.$route.query.hasOwnProperty('search')) {
            this.$store.commit('setGifs', res.data)
          }
        }
      })
      this.isLoading = false
    },
    async getSearchGifs(query = null) {
      this.page += 1
      if (query != null) {
        this.query = query
      }
      this.offset = this.page * this.limit - this.limit
      let url = `/gifs/search?api_key=${this.apiKey}&q=${this.splitSpace(this.query)}&limit=${this.limit}&offset=${this.offset}`
      await this.$axios.$get(url).then((res) => {
        if (res.meta.status === 200) {
          this.gifs = res.data
        }
      })
      this.$router.push({ query: { search: this.splitSpace(this.query) } })
      this.isLoading = false
    },
    async getPopulerSearchText() {
      await this.$axios.$get(`/trending/searches?api_key=${this.apiKey}`).then((res) => {
        if (res.meta.status === 200) {
          this.$store.commit('setPopulerSearchText', res.data)
          this.populerSearchText = res.data
        }
      })
    },

    getWindowHeight(event) {
      this.windowHeight = window.scrollY
      this.scrollToTopView = window.scrollY > 400 ? true : false
    },
  },
}
</script>
