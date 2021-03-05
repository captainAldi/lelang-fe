<template>
  <div>
    <!-- Alert Jika Gagal -->
    <v-alert 
      :type="alertObject.type"
      v-model="alertObject.status"
      dismissible
    >
      <li
        v-for="(value, key) of alertObject.message"
        :key="key"
      >
        <template v-for="item in value"> {{item}} </template>
      </li>
    </v-alert>

    <center>
      <v-progress-circular
        :size="70"
        :width="7"
        color="primary"
        indeterminate
      ></v-progress-circular>
    </center>

  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import axios from 'axios'

export default {
  name: 'verifyEmail',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
      rememberToken: ''
    }
  },
  mounted() {
    this.ambilDataUrl()
    this.verify()
  },
  computed: {
    ...mapGetters({
      user: 'auth/user',
    }),
  },
  methods: {
    ...mapActions({
      setAlert  : 'alert/set',
      setDialog : 'dialog/set',
      setAuth   : 'auth/set'
    }),

    ambilDataUrl() {
      this.rememberToken = this.$route.params
    },

    async verify() {
      try {

        this.setDialog({
          status : true,
        })

        await axios.get(this.api_url +'/verify-email/' + this.rememberToken.id)

        this.setDialog({
          status : false,
        })

        this.$router.push('/auth/login')

        this.setAlert({
          status : true,
          color  : 'success',
          text  : 'Verifikasi Email Berhasil, Silahkan Login !',
        })


      } catch (error) {

        this.setDialog({
          status : false,
        })

        if (error.response.status === 401) {
          this.setAlert({
            status : true,
            color  : 'error',
            text  : 'Email Tidak Terdaftar !',
          })
        } else {
          this.setAlert({
            status : true,
            color  : 'error',
            text  : 'Ada Masalah di Server !',
          })

          this.alertObject.status = true
          this.alertObject.type = 'error'
          this.alertObject.message = error.response.data

        }
      }
    }

  }
}
</script>

<style>

</style>