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
              <div class="icon-arrow">
                <i class="material-icons" color="blue lighten-5">
                keyboard_arrow_down
                </i>
                <v-card-title primary-title>
                  <h3 class="title-simulator">{{ title }}</h3>
                  <span class="grey--text subtitle-simulator">{{ subtitle }}</span>
                </v-card-title>
              </div>
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
      title: 'Simulador',
      subtitle: 'Na Ciclic seu sonho rende mais que na poupança',
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
  .icon-arrow i{
    font-size: 40px;
    font-weight: bold;
    color: #324376;
    margin-top: -10px;
  }
  .title-simulator {
    margin-top: -20px;
    font-family: 'Montserrat', sans-serif;
    color: #324376;
    font-size: 28px;
    font-weight: 500;
    text-transform: uppercase;
    text-align: center;
    margin-right: auto;
    margin-left: auto;
    padding: 7px 7px 0px 7px;
  }
  .v-card__title{
    padding: 0;
  }
  .subtitle-simulator{
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 30px;
    word-break: break-word;
  }
</style>
