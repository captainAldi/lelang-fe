<template>
  <div>
    <v-container fluid>
      <h1>Data Barang</h1>
      <v-card>
        <v-card-title>
          Barang
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
          :items="dataBarang"
          :search="search"
          :loading="tableLoading"
          mobile-breakpoint="0"
        >
          <template v-slot:item="row">
              <tr>
                <td>{{row.index + 1}}</td>
                <td>{{row.item.nama_barang}}</td>
                <td>{{row.item.detail_barang}}</td>
                <td>{{row.item.kondisi_barang}}</td>
                <td>{{row.item.harga_awal_barang}}</td>
                <td>
                  <section
                    v-if="row.item.photo_barang !== ''"
                  >
                    <v-img
                      :lazy-src="`${api_url}/files/photo-barang/${row.item.photo_barang.split('.')[0]}/${row.item.photo_barang.split('.')[1]}`"
                      :src="`${api_url}/files/photo-barang/${row.item.photo_barang.split('.')[0]}/${row.item.photo_barang.split('.')[1]}`"
                      max-width="250"
                      aspect-ratio="1"
                      alt="pic"
                      class="mb-5 mt-5 mx-auto"
                    ></v-img>
                  </section>

                  <section
                    v-else
                  >
                    <v-img
                      :lazy-src="require('../../../assets/aplikasi/noimage.png')"
                      :src="require('../../../assets/aplikasi/noimage.png')"
                      max-width="250"
                      alt="pic"
                      aspect-ratio="1"
                      class="mb-5 mt-5 mx-auto"
                    ></v-img>
                  </section>
                </td>
                <td>
                    <v-btn
                      small
                      class="mx-2"
                      icon
                      color="green" 
                      @click="detailData(row.item)"
                    >
                        <v-icon dark>mdi-information-outline</v-icon>
                    </v-btn>
                </td>
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
      search: '',
      tableLoading: false,
      headers: [
        { text: 'No', value: 'no', sortable: false },
        { text: 'Nama Barang', value: 'nama_barang' },
        { text: 'Detail', value: 'detail_barang' },
        { text: 'Kondisi', value: 'kondisi_barang' },
        { text: 'Harga Awal', value: 'harga_awal_barang' },
        { text: 'Photo', value: 'photo_barang', sortable: false  },
        { text: 'Actions', value: 'controls', sortable: false },
      ],
      dataBarang: []
    }
  },
  mounted() {
    this.getAllBarangs()
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
    
    async getAllBarangs() {
      try {

        this.tableLoading = true

        let config = {
          headers: {
            'Authorization': this.user.api_token
          }
        }

        let response = await axios.get(this.api_url + '/lelang/barangs/' + this.$route.params.kota, config)

        this.dataBarang = response.data.data

        this.tableLoading = false
      } catch (error) {
        console.log(error.response.message)
      }
    },

    detailData(e) {
      this.$router.push(`${this.$route.params.kota}/${e.kode_barang}`)
    },

  }
}
</script>

<style>

</style>