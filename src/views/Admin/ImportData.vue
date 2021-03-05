<template>
  <div>
    <v-container fluid>
      <h1>Import Data</h1>
      
      <!-- Barang -->
      <v-card
        class="mx-auto mb-5"
        max-width="800"
      >
        <v-card-title>
          <strong>Barang</strong>
        </v-card-title>

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

        <v-img
          :lazy-src="require('../../assets/aplikasi/contohImportBarang.png')"
          :src="require('../../assets/aplikasi/contohImportBarang.png')"
          
          alt="pic"
          aspect-ratio="12"
          class="mb-5 mt-5 ml-2 mr-2 mx-auto"
        ></v-img>

        <v-card-text class="text--primary">
          <v-form
            ref="formImportBarang"
            v-model="validImportBarangForm"
            lazy-validation
          >
            <v-file-input
              chips
              label="File Excel"
              @change="logFileBarang"
              filled
              class="mb-2"
              :rules="importRules"
              show-size
              accept=".xls,.xlsx"
            >
            </v-file-input>
          </v-form>
        </v-card-text>

        <v-card-actions>
          <v-btn
            :disabled="!validImportBarangForm"
            color="success"
            class="mr-4"
            @click="submitBarang"
          >
            Add
          </v-btn>

          <v-btn
            color="error"
            class="mr-4"
            @click="resetBarang"
          >
            Reset
          </v-btn>
        </v-card-actions>
      </v-card>

    </v-container>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import axios from 'axios'


export default {
  name: 'ImportData',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validImportBarangForm: false,

      fileDataBarang: null,

      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
    }
  },
  mounted() {
    
  },
  computed: {
    ...mapGetters({
      user  : 'auth/user',
      guest : 'auth/guest'
    }),

    importRules() {
      let rules

      if(this.fileData != null) {
        rules =  [
          v => !!v || 'File is required',
          v => !v || v.size < (3072 * 1024) || 'File Terlalu Besar > 3 MB !',
          v => !v || /^\S+$/.test(v.name) || 'Nama File Tidak Boleh ada Spasi'
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

    logFileBarang(e) {
      this.fileDataBarang = e
    },
    
    async submitBarang(e) {
      e.preventDefault()

      if (!this.$refs.formImportBarang.validate()) {
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
              'Content-Type': 'multipart/form-data',
            }, 
          }

          let formData = new FormData()

          formData.append('file_barang', this.fileDataBarang)
          //formData.append('fileName', this.fileData.name)


          const response = await axios.post(this.api_url + '/admin/barang/import', formData, config)

          this.setDialog({
            status : false,
          })

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })
          

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

    resetBarang() {
      this.$refs.validImportBarangForm.resetValidation()
    },


  }
}
</script>

<style>

</style>