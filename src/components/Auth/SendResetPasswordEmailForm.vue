<template>
  <div>
    <!-- Alert Jika Gagal -->
    <v-alert 
      :type="alertObject.type"
      v-model="alertObject.status"
      dismissible
    >
      {{ alertObject.message }}
    </v-alert>

    <v-form
      ref="formResetPasswordEmail"
      v-model="validResetPasswordEmail"
      lazy-validation
    >
      <v-text-field
        v-model="form.email"
        label="E-Mail ..."
        :rules="formRules.emailRules"
        filled
        required
      ></v-text-field>

      <v-btn
        :disabled="!validResetPasswordEmail"
        color="success"
        class="mr-4"
        @click="submit"
      >
        Verify
      </v-btn>

      <v-btn
        color="error"
        class="mr-4"
        @click="reset"
      >
        Reset
      </v-btn>
    </v-form>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import axios from 'axios'

export default {
  name: 'ResetPasswordEmailForm',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validResetPasswordEmail: false,
      showPassTextInput: false,
      form: {
        email: '',
      },
      formRules: {
        emailRules: [
          v => !!v || 'E-mail is required',
          v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
        ],
      },
      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
    }
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

    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formResetPasswordEmail.validate()) {
        this.setAlert({
          status : true,
          color  : 'error',
          text  : 'Isi Data dengan Lengkap !',
        })
      } else {

        try {

          this.setDialog({
            status : true,
          })

          let formData = new FormData()

          formData.append('email', this.form.email)

          await axios.post(this.api_url + '/send-reset-password-email', formData)

          this.setDialog({
            status : false,
          })

          this.$router.push('/auth/login')

          this.setAlert({
            status : true,
            color  : 'success',
            text  : 'Check Your E-Mail !',
          })

        } catch (error) {

          this.setDialog({
            status : false,
          })

          if (error.response.status === 401) {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : 'Anda Belum Ter Daftar !',
            })
          } else {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : 'Ada Masalah di Server !',
            })

            this.alertObject.status = true
            this.alertObject.type = 'error'
            this.alertObject.message = error.response.data.message

          }

        }
        
      }
    },
    
    reset() {
      this.$refs.formResetPasswordEmail.resetValidation()
      
      this.form.name = ''
      this.form.email = ''
      this.form.password = ''
      
    },
  }
}
</script>

<style>

</style>