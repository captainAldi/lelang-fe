<template>
  <div>
    <!-- Alert Jika Gagal -->
    <v-alert 
      :type="alertObject.type"
      v-model="alertObject.status"
      dismissible
    >
      <li>
        <template v-for="item in alertObject.message"> {{item}} </template>
      </li>
    </v-alert>

    <v-form
      ref="formRegister"
      v-model="validRegister"
      lazy-validation
    >
      <v-text-field
        v-model="form.name"
        label="Name ..."
        :rules="formRules.nameRules"
        filled
        required
        class="mt-5"
      ></v-text-field>

      <v-text-field
        v-model="form.email"
        label="E-Mail ..."
        :rules="formRules.emailRules"
        filled
        required
      ></v-text-field>

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
        :disabled="!validRegister"
        color="success"
        class="mr-4"
        @click="submit"
      >
        Register
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
  name: 'RegisterForm',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validRegister: false,
      showPassTextInput: false,
      form: {
        name: '',
        email: '',
        password: ''
      },
      formRules: {
        nameRules: [
          v => !!v || 'Name is required',
        ],
        emailRules: [
          v => !!v || 'E-mail is required',
          v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
        ],
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
  methods: {
    ...mapActions({
      setAlert  : 'alert/set',
      setDialog : 'dialog/set',
      setAuth   : 'auth/set'
    }),

    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formRegister.validate()) {
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

          formData.append('name', this.form.name)
          formData.append('email', this.form.email.toLowerCase())
          formData.append('password', this.form.password)

          await axios.post(this.api_url + '/register', formData)

          this.setDialog({
            status : false,
          })

          this.$router.push('/auth/login')

          this.setAlert({
              status : true,
              color  : 'success',
              text  : 'Register Berhasil, Verifikasi Email Anda !',
            })

        } catch (error) {

          console.log(error.response.data)

          this.setDialog({
            status : false,
          })

          if (error.response.status === 406) {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : error.response.data.messages,
            })
          } else {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : 'Ada Masalah di Server !',
            })

            this.alertObject.status = true
            this.alertObject.type = 'error'
            this.alertObject.message = error.response.data.messages
          }

        }
        
      }
    },
    
    reset() {
      this.$refs.formRegister.resetValidation()
      
      this.form.name = ''
      this.form.email = ''
      this.form.password = ''
      
    },
  }
}
</script>

<style>

</style>