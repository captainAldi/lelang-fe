<template>
  <div>
    <v-container fluid>
      <h1>Edit Data Barang</h1>

      <v-card>
        <v-card-title>
          Edit
        </v-card-title>

        <v-card-text>
          <EditBarangForm/>
        </v-card-text>

      </v-card>
    </v-container>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'

export default {
  name: 'editBarang',
  components: {
    EditBarangForm: () => import( /* webpackChunkName: "editBarangForm" */ '@/components/Admin/Barang/EditForm.vue'),
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