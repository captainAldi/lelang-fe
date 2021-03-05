<template>
  <div>
    <v-container fluid>
      <h1>Tambah Data Barang</h1>

      <v-card>
        <v-card-title>
          Tambah
        </v-card-title>

        <v-card-text>
          <TambahBarangForm/>
        </v-card-text>

      </v-card>
    </v-container>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'

export default {
  name: 'tambahBarang',
  components: {
    TambahBarangForm: () => import( /* webpackChunkName: "tambahBarangForm" */ '@/components/Admin/Barang/AddForm.vue'),
  },
  data() {
    return {

    }
  },
  mounted() {

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

    
    
  }
}
</script>

<style>

</style>