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

        <v-btn
          dark
          small
          class="ma-2"
          color="success" 
          to="/admin/data-barang/create"
        >
          <v-icon dark>mdi-plus-box-multiple-outline</v-icon>
        </v-btn>

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
                      color="pink" 
                      @click="editData(row.item)"
                    >
                        <v-icon dark>mdi-circle-edit-outline</v-icon>
                    </v-btn>

                    <v-btn
                      small
                      class="mx-2"
                      icon
                      color="pink" 
                      @click="deleteData(row.item)"
                    >
                        <v-icon dark>mdi-trash-can-outline</v-icon>
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

        let response = await axios.get(this.api_url + '/admin/barangs', config)

        this.dataBarang = response.data.data

        this.tableLoading = false
      } catch (error) {
        console.log(error.response.message)
      }
    },

    editData(e) {
      this.$router.push('data-barang/edit/' + e.kode_barang)
    },

    async deleteData(e) {
      const sw = await this.$swal.fire({
        title: 'Are you sure?',
        text: `Delete "${e.kode_barang}" ?`,
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
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

          const response = await axios.delete(this.api_url + '/admin/barang/delete/' + e.kode_barang, config)

          this.setDialog({
            status : false,
          })

          this.getAllBarangs()

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