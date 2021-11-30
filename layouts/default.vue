<template>
  <div>
    <header class="header">
      <!--  Navbar -->
      <div class="container is-fluid">
        <nav class="navbar">
          <div class="navbar-left">
            <nuxt-link to="/" class="logo">
              <img src="~/assets/img/logo.svg" alt="S'GIPHY ICON" />
            </nuxt-link>
            <span class="line-border mx-3"></span>
            <a href="https://giphy.com/" class="giphy-link" target="_blank" ref="noopener noreferrer"> Powered By GIPHY </a>
          </div>
          <div class="navbar-right">
            <div class="social-media">
              <a :href="socialMedia.github" ref="noopener noreferrer" target="_blank">
                <b-icon icon="github"></b-icon>
              </a>
              <a :href="socialMedia.linkedin" ref="noopener noreferrer" target="_blank">
                <b-icon icon="linkedin"> </b-icon>
              </a>
              <a :href="socialMedia.instagram" ref="noopener noreferrer" target="_blank">
                <b-icon icon="instagram"></b-icon>
              </a>
              <a :href="socialMedia.twitter" ref="noopener noreferrer" target="_blank">
                <b-icon icon="twitter"></b-icon>
              </a>
            </div>
          </div>
        </nav>
      </div>
    </header>
    <!--  Main -->
    <main>
      <!--  Child-page -->
      <nuxt />
    </main>
  </div>
</template>

<script>
export default {
  name: 'MainLayout',
  computed: {
    socialMedia() {
      return this.$store.state.socialMedia
    },
  },
}
</script>
