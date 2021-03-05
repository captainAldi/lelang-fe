<template>
  <div>
    <v-row
      align="center"
      justify="center"
    >
      <v-col
        cols="12"
        lg="6"
        md="8"
        align-self="center"
      >
        <v-img
          :lazy-src="require('@/assets/aplikasi/logo.png')"
          :src="require('@/assets/aplikasi/logo.png')"
          max-width="150"
          max-height="150"
          class="mt-5 mx-auto"
        ></v-img>
      </v-col>
    </v-row>

    <!-- Form -->
    <v-row
      align="center"
      justify="center"
    >
      <v-col
        cols="12"
        lg="6"
        md="8"
        align-self="center"
      >        

        <v-card
          elevation="4"
          class="mx-auto mt-5 "
          max-width="80%"
        >
          <v-card-title class="justify-center">Register Page</v-card-title>
          <v-divider></v-divider>
          <v-card-text>
            <RegisterForm/>
          </v-card-text>
          <v-card-subtitle>            
            <router-link
              to="/auth/login"
            >
              Login
            </router-link>
          </v-card-subtitle>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import RegisterForm from '../../components/Auth/RegisterForm'

export default {
  name: 'Register',
  components: {
    RegisterForm
  },
  data() {
    return {

    }
  },
  methods: {

  }
}
</script>

<style>

</style>