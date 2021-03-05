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
      ref="formEditProfile"
      v-model="validEditProfileForm"
      lazy-validation
    >
      <v-text-field
        v-model="form.name"
        label="Nama ..."
        :rules="formRules.nameRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <v-text-field
        v-model="form.email"
        label="E-Mail ..."
        :rules="formRules.emailRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <span>Jika Tidak Ingin di Ubah Biarkan Kosong</span>
      <v-text-field
        label="Password ..."
        v-model="password"
        :rules="passwordRules"
        :append-icon="showPassTextInput ? 'mdi-eye' : 'mdi-eye-off'"
        :type="showPassTextInput ? 'text' : 'password'"
        @click:append="showPassTextInput = !showPassTextInput"
        filled
        required
        class="mb-2"
      ></v-text-field>
      

      <v-btn
        :disabled="!validEditProfileForm"
        color="success"
        class="mr-4"
        @click="submit"
      >
        Edit
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
  name: 'editProfileForm',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validEditProfileForm: false,
      showPassTextInput: false,
      idProfile: '',
      currentPhotoProfile: '',
      form: {
        name: '',
        email: '',
      },
      password: null,
      formRules: {
        emailRules: [
          v => !!v || 'E-mail is required',
          v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
        ],
        nameRules: [
          v => !!v || 'Nama is required',
        ],
      },
      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
    }
  },
  mounted() {
    this.ambilDataUrl()
    this.getCurrentData()
  },
  computed: {
    ...mapGetters({
      user  : 'auth/user',
      guest : 'auth/guest'
    }),

    passwordRules() {
      let rules

      if(this.password) {
        rules =  [
          v => /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d~`!$%@#£€*?&^()<>,.\-+=_|/;:'"{}[\s]{6,}$/.test(v) || "Use At Least 6 Characters, with Letter and Number"
        ]
      } else {
        rules = []
      }

      return rules
    },

  },
  methods: {
    ...mapActions({
      setAlert  : 'alert/set',
      setAuth   : 'auth/set',
      setDialog : 'dialog/set'
    }),


    ambilDataUrl() {
      this.idProfile = this.$route.params.id
    },

    async getCurrentData() {
      try {
        this.setDialog({
          status : true,
        })
        
        let config = {
          headers: {
            'Authorization': this.user.api_token,
          }
        }

        const response = await axios.get(this.api_url + '/profile/' + this.idProfile, config)

        this.form = response.data.data

        this.setDialog({
          status : false,
        })

      } catch (error) {
        this.setDialog({
          status : false,
        })
        
        this.setAlert({
          status : true,
          color  : 'error',
          text  : error.response.data.message,
        })
      }
    },


    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formEditProfile.validate()) {
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
          
          let config = {
            headers: {
              'Authorization': this.user.api_token,
            }, 
          }

          let formData = new FormData()

          formData.append('email', this.form.email.toLowerCase())
          formData.append('name', this.form.name)
          formData.append("_method", "PATCH");

          if(this.password != null) {
            formData.append('password', this.password)
          }

          const response = await axios.post(this.api_url + '/profile/update/' + this.idProfile, formData, config)

          this.setAuth(response.data.result)

          this.setDialog({
            status : false,
          })

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })

          this.$router.go('/')
          

        } catch (error) {
          this.setDialog({
            status : false,
          })

          if (error.response.status === 422) {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : 'Inputan tidak Valid !',
            })
          } else {
            this.setAlert({
              status : true,
              color  : 'error',
              text  : 'Internal Server Error !',
            })
          }

          this.alertObject.status = true
          this.alertObject.type = 'error'
          this.alertObject.message = error.response.data
          
        }
      }

    },

    reset() {
      this.$refs.formEditProfile.resetValidation()
      
      this.form.email = ''
      this.password = ''
      this.form.name = ''
      
    },
    
  }
}
</script>

<style>

</style>