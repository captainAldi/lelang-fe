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
      ref="formResetPassword"
      v-model="validResetPassword"
      lazy-validation
    >
      <v-text-field
        label="Password ..."
        v-model="form.password"
        :rules="formRules.passwordRules"
        :append-icon="showPassTextInput ? 'mdi-eye' : 'mdi-eye-off'"
        :type="showPassTextInput ? 'text' : 'password'"
        @click:append="showPassTextInput = !showPassTextInput"
        filled
        required
      ></v-text-field>

      <v-btn
        :disabled="!validResetPassword"
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
  name: 'ResetPasswordForm',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validResetPassword: false,
      showPassTextInput: false,
      rememberToken: '',
      form: {
        password: '',
      },
      formRules: {
        passwordRules: [
          v => !!v || 'Password is required',
          v => /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d~`!$%@#£€*?&^()<>,.\-+=_|/;:'"{}[\s]{6,}$/.test(v) || "Use At Least 6 Characters, with Letter and Number"
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
  mounted() {
    this.ambilDataUrl()
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

    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formResetPassword.validate()) {
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

          formData.append('password', this.form.password)

          await axios.post(this.api_url + '/reset-password/' + this.rememberToken.id, formData)

          this.setDialog({
            status : false,
          })

          this.$router.push('/auth/login')

          this.setAlert({
            status : true,
            color  : 'success',
            text  : 'Reset Password Berhasil !',
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
      this.$refs.formResetPassword.resetValidation()
      
      this.form.password = ''
      
    },
  }
}
</script>

<style>

</style>