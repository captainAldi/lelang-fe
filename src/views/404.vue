<template>
  <v-app class="app">
    <Alert />
    <Dialog />

    <v-main>
      <v-container>
        <v-slide-y-transition mode="out-in">
          
          <v-row
            align="center"
            justify="center"
          >
            <v-col
              cols="12"
              lg="6"
              md="8"
              align-self="center"
            >
              <v-card
                elevation="4"
                class="mx-auto mt-5 "
                max-width="80%"
              >
                <v-card-title class="justify-center">Not Found</v-card-title>

                <v-divider></v-divider>
                
                <center>
                  <v-progress-circular
                    :size="70"
                    :width="7"
                    color="primary"
                    class="mt-5"
                    indeterminate
                  ></v-progress-circular>
                </center>

                <v-card-text>
                  
                </v-card-text>
                <v-card-subtitle>
                  Oops, <strong>404</strong> 
                  <br>
                  Nothings Found Here ...
                  <br>
                  You'll be redirected soon
                </v-card-subtitle>
              </v-card>
            </v-col>
          </v-row>

        </v-slide-y-transition>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'NotFound',

  components: {
    Alert: () => import( /* webpackChunkName: "alert" */ '@/components/Alert.vue'),
    Dialog: () => import( /* webpackChunkName: "dialogLoading" */ '@/components/Dialog.vue'),
  },

  mounted() {
    this.redirectToHome()
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

  },

  methods: {
    redirectToHome(){
      setTimeout(() => {
        let checkAdmin = this.user.profile.role == 'Admin'

        console.log

        if (checkAdmin) {
          this.$router.push('/admin/home')
        } else {
          this.$router.push('/user/home')
        }
      }, 3000)
    }
  }


}
</script>

<style scoped>
.app {
  background-color: #54c2cc;
  height: 100%;
}
</style>