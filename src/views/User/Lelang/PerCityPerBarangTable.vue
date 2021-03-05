<template>
  <div>
    <v-container fluid>
      <h1>Detail Barang</h1>

      <v-card
        class="mx-auto mb-5"
        max-width="800"
      >
        <section v-if="currentPhotoBarang != ''">
          <v-img
            class="white--text align-end"
            height="200px"
            :src="currentPhotoBarang"
            :lazy-src="currentPhotoBarang"
          >
            <v-card-title>Top 10 Australian beaches</v-card-title>
          </v-img>
        </section>

        <section v-else>
          <v-img
            :lazy-src="require('../../../assets/aplikasi/noimage.png')"
            :src="require('../../../assets/aplikasi/noimage.png')"
            max-width="250"
            alt="pic"
            aspect-ratio="1"
            class="mb-5 mt-5 mx-auto"
          ></v-img>
        </section>

        <v-card-subtitle class="pb-0">
          <strong>Kode Barang : </strong>{{dataBarang.kode_barang}}
        </v-card-subtitle>

        <v-card-text class="text--primary">
          <div><strong>Nama Barang : </strong>{{ dataBarang.nama_barang }}</div>

          <div><strong>Harga : </strong>{{ dataBarang.harga_awal_barang }}</div>
        </v-card-text>

        <v-card-actions>
          <v-btn
            color="orange"
          >
            Share
          </v-btn>
        </v-card-actions>
      </v-card>

      <v-card
        class="mx-auto mb-5"
        max-width="800"
      >
        <v-card-title>
          <strong>Place a New Bid</strong>
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

        <v-card-text class="text--primary">
          <v-form
            ref="formAddPenawaran"
            v-model="validAddPenawaranForm"
            lazy-validation
          >
            <v-text-field
              v-model="form.harga_penawaran"
              label="Harga Penawaran ..."
              :rules="formRules.hargaPenawaranRules"
              filled
              type="number"
              required
              class="mb-2"
            ></v-text-field>
          </v-form>
        </v-card-text>

        <v-card-actions>
          <v-btn
            :disabled="!validAddPenawaranForm"
            color="success"
            class="mr-4"
            @click="submit"
          >
            Add
          </v-btn>

          <v-btn
            color="error"
            class="mr-4"
            @click="reset"
          >
            Reset
          </v-btn>
        </v-card-actions>
      </v-card>

      <v-card
        class="mx-auto mb-5"
        max-width="800"
      >
        <v-card-title>
          Lelang
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="dataLelang"
          :search="search"
          :loading="tableLoading"
          mobile-breakpoint="0"
        >
          <template v-slot:item="row">
              <tr>
                <td>{{row.index + 1}}</td>
                <td>{{row.item.harga_tawaran}}</td>
                <td>{{row.item.status_lelang}}</td>
              </tr>
          </template>
        </v-data-table>

      </v-card>
    </v-container>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import axios from 'axios'
import 'vue-sweetalert2';


export default {
  name: 'DataBarang',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      dataBarang: [],
      validAddPenawaranForm: false,
      form: {
        harga_penawaran: ''
      },
      formRules: {
        hargaPenawaranRules: [
          v => !!v || 'Harga Penawaran is required',
        ],
      },
      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
      currentPhotoBarang: '',

      search: '',
      tableLoading: false,
      headers: [
        { text: 'No', value: 'no', sortable: false },
        { text: 'Nilai Tawaran', value: 'harga_tawaran'  },
        { text: 'Status', value: 'status_lelang'  },
      ],
      dataLelang: []
    }
  },
  mounted() {
    this.getCurrentData()
    this.getCurrentLelang()
  },
  computed: {
    ...mapGetters({
      user  : 'auth/user',
      guest : 'auth/guest'
    }),
  },
  methods: {
    ...mapActions({
      setAlert  : 'alert/set',
      setAuth   : 'auth/set',
      setDialog : 'dialog/set'
    }),
    
    async getCurrentData() {
      try {

        this.setDialog({
          status : true,
        })

        let config = {
          headers: {
            'Authorization': this.user.api_token
          }
        }

        let response = await axios.get(this.api_url + '/lelang/barangs/specific/' + this.$route.params.kodeBarang, config)

        this.dataBarang = response.data.data[0]

        if(this.dataBarang.photo_barang) {
          this.getCurrentPhotoBarang()
        }

        console.log(this.dataBarang)

        this.setDialog({
          status : false,
        })

      } catch (error) {
        console.log(error.response.message)
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

    async getCurrentLelang() {
      try {

        this.tableLoading = true

        let config = {
          headers: {
            'Authorization': this.user.api_token
          }
        }

        let response = await axios.get(this.api_url + '/lelang/penawaran/list/' +  this.$route.params.kodeBarang, config)

        this.dataLelang = response.data.data

        this.tableLoading = false
      } catch (error) {
        console.log(error.response.message)
      }
    },

    async submit(e) {
      e.preventDefault()

      if (!this.$refs.formAddPenawaran.validate()) {
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

          formData.append('id_barang', this.dataBarang.id)
          formData.append('harga_tawaran', this.form.harga_penawaran)

          const response = await axios.post(this.api_url + '/lelang/penawaran/create', formData, config)

          this.setDialog({
            status : false,
          })

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })

          await this.getCurrentData()
          this.reset()
          

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
      this.$refs.formAddPenawaran.resetValidation()

      this.form.harga_penawaran = ''
      
    },
  }
}
</script>

<style>

</style>