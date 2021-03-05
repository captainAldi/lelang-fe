<template>
  <div>
    <v-container fluid>
      <h1>Pilih Kota</h1>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title>
              Jakarta
            </v-card-title>

            <v-card-actions>
              <v-btn
                color="orange"
                @click="goToJakarta"
              >
                Choose
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>

        <v-col>
          <v-card>
            <v-card-title>
              Malang
            </v-card-title>

            <v-card-actions>
              <v-btn
                color="orange"
                @click="goToMalang"
              >
                Choose
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'

export default {
  name: 'DataBarang',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
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
    
    goToMalang() {
      this.$router.push('data-lelang/Malang')
    },

    goToJakarta() {
      this.$router.push('data-lelang/Jakarta')
    }

  }
}
</script>

<style>

</style>