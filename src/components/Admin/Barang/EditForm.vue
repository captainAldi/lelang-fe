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
      ref="formEditBarang"
      v-model="validEditBarangForm"
      lazy-validation
    >
      <v-text-field
        v-model="form.kode_barang"
        label="Kode Barang ..."
        :rules="formRules.kodeBarangRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <v-text-field
        v-model="form.nama_barang"
        label="Nama Barang ..."
        :rules="formRules.namaBarangRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <v-text-field
        v-model="form.detail_barang"
        label="Detail Barang ..."
        :rules="formRules.detailBarangRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <v-text-field
        v-model="form.harga_awal_barang"
        label="Harga Awal Barang ..."
        :rules="formRules.hargaAwalBarangRules"
        filled
        type="number"
        required
        class="mb-2"
      ></v-text-field>

      <v-text-field
        v-model="form.kondisi_barang"
        label="Kondisi Barang ..."
        :rules="formRules.kondisiBarangRules"
        filled
        required
        class="mb-2"
      ></v-text-field>

      <v-select
        :items="['Jakarta', 'Malang', 'Jambi']"
        v-model="form.lokasi_barang"
        label="Solo field"
        dense
        solo
      ></v-select>

      <v-select
        :items="['Belum Terjual', 'Terjual']"
        v-model="form.status_barang"
        label="Solo field"
        disabled
        dense
        solo
      ></v-select>

      <section
        v-if="currentPhotoBarang != ''"
      >
        <v-img
          :lazy-src="currentPhotoBarang"
          :src="currentPhotoBarang"
          max-width="250"
          aspect-ratio="1"
          alt="pic"
          class="mb-2 mx-auto"
        ></v-img>

        <center>
          <span>Jika Tidak Ingin di Ubah Biarkan Kosong</span>
        </center>
      </section>

      <section
        v-else
      >
        <v-img
          :lazy-src="require('@/assets/aplikasi/noimage.png')"
          :src="require('@/assets/aplikasi/noimage.png')"
          max-width="250"
          alt="pic"
          aspect-ratio="1"
          class="mb-2 mx-auto"
        ></v-img>

      </section>

      <v-file-input
        chips
        label="Photo Barang"
        @change="logFile"
        filled
        class="mb-2"
        :rules="photoBarangRules"
        show-size
        accept=".png,.jpg,.jpeg"
      >
      </v-file-input>
      

      <v-btn
        :disabled="!validEditBarangForm"
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
  name: 'editGuruForm',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      validEditBarangForm: false,
      kodeBarang: '',
      currentPhotoBarang: '',
      form: {
        kode_barang: '',
        nama_barang: '',
        detail_barang: '',
        harga_awal_barang: '',
        kondisi_barang: '',
        lokasi_barang: '',
        status_barang: '',
      },
      fileData: null,
      formRules: {
        kodeBarangRules: [
          v => !!v || 'Kode Barang is required',
        ],
        namaBarangRules: [
          v => !!v || 'Nama Barang is required',
        ],
        detailBarangRules: [
          v => !!v || 'Detail Barang is required',
        ],
        hargaAwalBarangRules: [
          v => !!v || 'Harga Awal Barang is required',
        ],
        kondisiBarangRules: [
          v => !!v || 'Harga Awal Barang is required',
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

      if(this.form.password) {
        rules =  [
          v => /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d~`!$%@#£€*?&^()<>,.\-+=_|/;:'"{}[\s]{6,}$/.test(v) || "Use At Least 6 Characters, with Letter and Number"
        ]
      } else {
        rules = []
      }

      return rules
    },

    photoBarangRules() {
      let rules

      if(this.fileData != null) {
        rules =  [
          v => !!v || 'Photo Barang is required',
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

    logFile(e) {
      this.fileData = e
    },

    ambilDataUrl() {
      this.kodeBarang = this.$route.params.id
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

        const response = await axios.get(this.api_url + '/admin/barang/specific/' + this.kodeBarang, config)

        this.form = response.data.data

        if(this.form.photo_barang) {
          this.getCurrentPhotoBarang()
        }

        this.setDialog({
          status : false,
        })

      } catch (error) {
        this.setAlert({
          status : true,
          color  : 'error',
          text  : error.response.data.message,
        })
      }
    },

    async getCurrentPhotoBarang() {

      let completeFileName = this.form.photo_barang

      if (completeFileName != '') {
        let fileName  = completeFileName.split('.').shift()
        let fileExt = completeFileName.split('.').pop()

        let newUrlPP = `${this.api_url}/files/photo-barang/${fileName}/${fileExt}`

        this.currentPhotoBarang = newUrlPP
      } else {
          this.currentPhotoBarang = ''
      }    
    },

    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formEditBarang.validate()) {
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

          formData.append('kode_barang', this.form.kode_barang)
          formData.append('nama_barang', this.form.nama_barang)
          formData.append('detail_barang', this.form.detail_barang)
          formData.append('harga_awal_barang', this.form.harga_awal_barang)
          formData.append('kondisi_barang', this.form.kondisi_barang)
          formData.append('lokasi_barang', this.form.lokasi_barang)
          formData.append('status_barang', this.form.status_barang)
          formData.append("_method", "PATCH");

          if(this.fileData !== null) {
            formData.append('photo_barang', this.fileData)
            formData.append('fileName', this.fileData.name)
          } 

          const response = await axios.post(this.api_url + '/admin/barang/update/' + this.kodeBarang, formData, config)

          this.setDialog({
            status : false,
          })

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })

          this.getCurrentData()
          

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
      this.$refs.formEditBarang.resetValidation()
      
      this.form.kode_barang = ''
      this.form.nama_barang = ''
      this.form.detail_barang = ''
      this.form.harga_awal_barang = ''
      this.form.kondisi_barang = ''
      this.form.lokasi_barang = ''
      
    },
    
  }
}
</script>

<style>

</style>