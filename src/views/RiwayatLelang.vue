<template>
  <div>
    <v-container fluid>
      <h1>Data Lelang</h1>
      <v-card>
        <v-card-title>
          Riwayat Lelang
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
                <td>{{row.item.barang.kode_barang}}</td>
                <td>{{row.item.barang.nama_barang}}</td>
                <td>{{row.item.user.name}}</td>
                <td>{{row.item.user.email}}</td>
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
      search: '',
      tableLoading: false,
      headers: [
        { text: 'No', value: 'no', sortable: false },
        { text: 'Kode Barang', value: 'barang.kode_barang' },
        { text: 'Nama Barang', value: 'barang.nama_barang' },
        { text: 'Penawar', value: 'user.name' },
        { text: 'E-Mail', value: 'user.email' },
        { text: 'Nilai Tawaran', value: 'harga_tawaran'  },
        { text: 'Status', value: 'status_lelang'  },
      ],
      dataLelang: []
    }
  },
  mounted() {
    this.getAllLelangs()
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
    
    async getAllLelangs() {
      try {

        this.tableLoading = true

        let config = {
          headers: {
            'Authorization': this.user.api_token
          }
        }

        let response = await axios.get(this.api_url + '/lelang/penawaran/list', config)

        this.dataLelang = response.data.data

        this.tableLoading = false
      } catch (error) {
        console.log(error.response.message)
      }
    },
    
  }
}
</script>

<style>

</style>