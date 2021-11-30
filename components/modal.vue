<template>
  <b-modal v-if="selectedGif" :active="isModal" :width="750" scroll="clip" :on-cancel="closeModal">
    <div class="card">
      <div class="card-content">
        <div class="columns is-multiline">
          <div class="column is-12-mobile is-4-tablet">
            <div class="user-box">
              <div class="columns is-multiline is-mobile mb-0" v-if="selectedGif.hasOwnProperty('user')">
                <div class="column is-3-mobile is-4-desktop">
                  <div class="user-profil-img">
                    <img :src="selectedGif.user.avatar_url" :alt="selectedGif.user.username" />
                  </div>
                </div>
                <div class="column is-9-mobile is-8-desktop">
                  <div class="user-profil-text">
                    <h6 class="display-name">{{ selectedGif.user.display_name }}</h6>
                    <div class="display-username">
                      <a :href="selectedGif.user.profile_url" target="_blank">@{{ selectedGif.user.username }} </a>
                      <b-icon class="icon" v-if="selectedGif.user.is_verified" icon="check-circle"></b-icon>
                    </div>
                  </div>
                </div>
              </div>
              <div class="user-description" v-if="selectedGif.hasOwnProperty('user')">
                <p class="mb-5 text-muted">{{ selectedGif.user.description }}</p>
              </div>
              <div class="social-media mb-5" v-if="selectedGif.hasOwnProperty('user')">
                <p class="mb-1">Sosyal Medya</p>
                <a
                  v-if="selectedGif.user.website_url.length > 0"
                  :href="selectedGif.user.website_url"
                  ref="nofollow noopener noreferrer"
                  target="_blank"
                >
                  <b-icon icon="web"></b-icon>
                </a>

                <a
                  v-if="selectedGif.user.instagram_url.length > 0"
                  :href="selectedGif.user.instagram_url"
                  ref="nofollow noopener noreferrer"
                  target="_blank"
                  ><b-icon icon="instagram"></b-icon
                ></a>
              </div>
              <div class="source-link mb-5" v-if="selectedGif.source.length > 0">
                <p>Kaynak</p>
                <a :href="selectedGif.source" ref="noopener noreferrer" target="_blank">{{
                  selectedGif.source_tld.length > 0 ? selectedGif.source_tld : selectedGif.source
                }}</a>
              </div>
              <div class="giphy-logo">
                <a href="https://giphy.com/" target="_blank" ref="nofollow noopener noreferrer">
                  <img src="~/assets/img/giphy-logo.png" alt="giphy-logo" />
                </a>
              </div>
            </div>
          </div>
          <div class="column is-12-mobile is-8-tablet">
            <div class="modal-media-box">
              <div class="modal-header mb-2">
                <div class="btn-group">
                  <button class="btn-link mx-1">
                    <b-icon icon="share"></b-icon>
                  </button>
                  <button class="btn-link btn-orange" @click="download(selectedGif.id, selectedGif.images.downsized.url)">
                    <b-icon icon="download"></b-icon>
                  </button>
                </div>
              </div>
              <div class="modal-media">
                <img :src="selectedGif.images.fixed_width.url" :alt="selectedGif.title" />
                <h2 class="title is-6">{{ selectedGif.title }}</h2>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </b-modal>
</template>

<script>
export default {
  name: 'Modal',
  props: ['selectedGif', 'isModal'],
  methods: {
    download(id, url) {
      this.$axios({
        url: url,
        method: 'GET',
        responseType: 'blob',
      }).then((response) => {
        var fileURL = window.URL.createObjectURL(new Blob([response.data]))
        var fileLink = document.createElement('a')

        fileLink.href = fileURL
        fileLink.setAttribute('download', id + '.gif')
        document.body.appendChild(fileLink)

        fileLink.click()
      })
    },
    closeModal() {
      this.$emit('update:isModal', !this.isModal)
    },
  },
}
</script>

<style></style>
