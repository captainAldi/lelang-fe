<template>
  <v-app class="app">
    <Alert />
    <Dialog />

    <v-main>
      <v-container>
        <v-slide-y-transition mode="out-in">
          <router-view></router-view>
        </v-slide-y-transition>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'admin-home',

  components: {
    Alert: () => import( /* webpackChunkName: "alert" */ '@/components/Alert.vue'),
    Dialog: () => import( /* webpackChunkName: "dialogLoading" */ '@/components/Dialog.vue'),
  },

  mounted() {
    this.getTahun()
    this.getThemeDataLS()
  },

  data: () => ({
    tahun: '',
    app_theme_key: process.env.VUE_APP_LS_THEME_KEY,
  }),

  computed: {
    ...mapGetters({
      user: 'auth/user',
      guest: 'auth/guest',
    }),

    goDark: {
      get() {
        const theme = JSON.parse(localStorage.getItem(this.app_theme_key))
        return theme
      },
      set(v) {
        this.$vuetify.theme.dark = v
        localStorage.setItem(this.app_theme_key, v)
      }
    },
  },

  methods: {

    getTahun() {
      this.tahun = new Date().getFullYear()
    },

    async logOutSSO() {
      await localStorage.clear()
      this.$router.go(0)
    },

    getThemeDataLS() {
      const theme = localStorage.getItem(this.app_theme_key);


      if (theme) {
          if (theme == "true") {
              this.$vuetify.theme.dark = true;
          } else {
              this.$vuetify.theme.dark = false;
          }
      }
    },

  }


}
</script>

<style scoped>
.app {
  background-color: #54c2cc;
  height: 100%;
}
</style>