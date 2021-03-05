<template>
  <div>
    <v-container fluid>
      <h1>Data Lelang</h1>
      <v-card>
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

        <!-- <v-btn
          dark
          small
          class="ma-2"
          color="success" 
          to="/admin/data-barang/create"
        >
          <v-icon dark>mdi-plus-box-multiple-outline</v-icon>
        </v-btn> -->

        <!-- <v-btn
          dark
          small
          color="red lighten-1" 
          @click="exportPDF"
          class="ma-2"
        >
          <v-icon dark>mdi-file-pdf-outline</v-icon>
        </v-btn> -->

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
                <td>
                  <router-link
                    :to="`/admin/data-barang/edit/${row.item.barang.kode_barang}`"
                  >
                    {{row.item.barang.kode_barang}}
                  </router-link>
                </td>
                <td>{{row.item.barang.nama_barang}}</td>
                <td>{{row.item.user.name}}</td>
                <td>{{row.item.user.email}}</td>
                <td>{{row.item.harga_tawaran}}</td>
                <td>{{row.item.status_lelang}}</td>
                <td>
                    <section v-if="row.item.barang.status_barang != 'Terjual'">
                      <v-btn
                        small
                        class="mx-2"
                        icon
                        color="green" 
                        @click="approveData(row.item)"
                      >
                          <v-icon dark>mdi-check-circle-outline</v-icon>
                      </v-btn>
                    </section>

                    <section v-else>
                      <v-btn
                        small
                        class="mx-2"
                        icon
                        color="green" 
                        @click="approveData(row.item)"
                        disabled
                      >
                          <v-icon dark>mdi-check-circle-outline</v-icon>
                      </v-btn>
                    </section>
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
        { text: 'Kode Barang', value: 'barang.kode_barang' },
        { text: 'Nama Barang', value: 'barang.nama_barang' },
        { text: 'Penawar', value: 'user.name' },
        { text: 'E-Mail', value: 'user.email' },
        { text: 'Nilai Tawaran', value: 'harga_tawaran'  },
        { text: 'Status', value: 'status_lelang'  },
        { text: 'Actions', value: 'controls', sortable: false },
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

        let response = await axios.get(this.api_url + '/admin/lelang/penawaran/all', config)

        this.dataLelang = response.data.data

        this.tableLoading = false
      } catch (error) {
        console.log(error.response.message)
      }
    },

    async approveData(e) {
      const sw = await this.$swal.fire({
        title: 'Are you sure?',
        //text: `Approve this Bid "${e.harga_tawaran}" ? <br/> for "${e.barang.nama_barang}"`,
        icon: 'question',
        html: `Approve this Bid <strong>"${e.harga_tawaran}"</strong> ? <br/> for <strong>"${e.barang.nama_barang} - ${e.barang.kode_barang}"</strong>`,
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes !'
      })
      
      if (sw.value) {
        try {
          
          this.setDialog({
            status : true,
          })
          
          let config = {
            headers: {
              'Authorization': this.user.api_token
            }
          }

          let formData = new FormData()

          formData.append("_method", "PATCH");

          const response = await axios.post(this.api_url + '/admin/lelang/penawaran/approve/' + e.id, formData, config)

          this.setDialog({
            status : false,
          })

          this.getAllLelangs()

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })
          

        } catch (error) {
          this.setAlert({
            status : true,
            color  : 'error',
            text  : error.response.data,
          })
        }
      } 

    },

    async exportPDF() {
      try {
        
        this.setDialog({
          status : true,
        })
        
        let config = {
          headers: {
            'Authorization': this.user.api_token,
          },
          responseType: 'blob',
        }

        const response = await axios.get(`${this.api_url}/admin/data/guru/cetak`, config)

        const downloadUrl = window.URL.createObjectURL(new Blob([response.data], {type: 'application/pdf'}));

        // Get Date Time
        let dateTimeNow = new Date()

        const link = document.createElement('a');
        link.href = downloadUrl;
        link.setAttribute('download', `data-guru-${dateTimeNow.getDate()}${dateTimeNow.getMonth()}${dateTimeNow.getFullYear()}-${dateTimeNow.getHours()}${dateTimeNow.getMinutes()}.pdf`); //any other name + extension
        document.body.appendChild(link);
        link.click();
        link.remove();  

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
          text  : error,
        })
      }
    }
  }
}
</script>

<style>

</style>