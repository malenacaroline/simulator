<template>
  <div>
    <v-app light>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md6 lg4>
            <v-card elevation-12 class="card">
              <v-img class="logo"
                aspect-ratio="4.75" :src='image.url'>
              </v-img>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-app>
  </div>
</template>

<script>
import { VMoney } from 'v-money'
import axios from 'axios'

export default {
  name: 'Simulator',
  data () {
    return {
      msg: 'Simulador',
      image: {
        url: require('../assets/img/ciclic.jpeg')
      }
    }
  }
}
</script>

<style scoped>
  .card {
    height: 510px;
    margin-top: 30px;
    margin-bottom: 23px;
  }
  .logo{
    transform: scale(0.6);
  }
</style>
