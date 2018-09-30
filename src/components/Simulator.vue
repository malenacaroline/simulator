<template>
  <div>
    <v-app light>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md6 lg4>
            <v-card elevation-12 class="card">
              <!-- Head Card -->
              <v-img class="logo"
                aspect-ratio="4.75" :src='image.url'>
              </v-img>
              <div class="icon-arrow">
                <i class="material-icons" color="blue lighten-5">
                  keyboard_arrow_down
                </i>
              </div>
              <v-card-title primary-title>
                <h3 class="title-simulator">{{ title }}</h3>
                <span class="grey--text subtitle-simulator">{{ subtitle }}</span>
              </v-card-title>
              <!-- Form Data -->
              <v-form class="form" ref="form" v-model="valid" v-if="!result && !loading" lazy-validation>
                <!-- Input Name -->
                <v-text-field
                  v-model="user.name"
                  :rules="nameRules"
                  label="Nome"
                  @keyup = onlyLetter()>
                </v-text-field>
                <!-- Input Monthly Payment -->
                <v-text-field
                  v-model="user.monthlyPayment"
                  v-money="money"
                  label="Mensalidade"
                  :rules="monthlyRules">
                </v-text-field>
                <!-- Input Time -->
                <span>
                  <font class="label-start-time">Tempo</font><br>
                </span>
                <v-flex d-inline-flex class="flex-time">
                  <v-slider class="slider"
                    v-model="user.timeSimulator"
                    color="deep-purple darken-4"
                    min="1"
                    max="10"
                  ></v-slider>
                  <font class="time-simulator" v-if="user.timeSimulator == 1">({{ user.timeSimulator }} ano)</font>
                  <font class="time-simulator" v-if="user.timeSimulator > 1">({{ user.timeSimulator }} anos)</font>
                </v-flex>
                <!-- Btn Submit -->
                <v-btn round color="green accent-3" dark @click="submit()">
                  <i class="material-icons">mood</i>&nbsp;&nbsp;Simular
                </v-btn>
              </v-form>
              <!-- Screen Loading -->
              <div v-if="!result && loading" class="loading" align-center justify-center>
                <v-flex>
                  <v-progress-circular indeterminate color="primary"></v-progress-circular>
                </v-flex>
              </div>
              <!-- Screen with result the API -->
              <div v-if="result && !loading" class="result">
                <v-flex d-flex xs12>
                  <v-card class="card-result">
                    <div class="content-result">
                      <div class="icon-result">
                        <i class="material-icons">done</i>
                      </div>
                      <h2 class="title-result"> Olá {{ user.name }},</h2>
                      <h2 class="text-result">Juntando {{ user.monthlyPayment }} todo mês,
                        você terá R$ {{ user.valueResult }} em {{ user.timeSimulator }}</h2>
                    </div>
                  </v-card>
                </v-flex>
              </div>
              <v-btn v-if="result && !loading" round color="primary" dark @click="reset()">
                <i class="material-icons">mood</i>&nbsp;&nbsp;Simular Novamente
              </v-btn>
              <!-- Screen when Requisition failed-->
              <div v-if="reqFailed" class="result">
                <v-flex d-flex xs12>
                  <v-card class="card-result">
                    <div class="content-result">
                      <div class="icon-result">
                        <i class="material-icons">done</i>
                      </div>
                      <h2 class="title-result"> Ops <font class="username">{{ user.name }}</font>!</h2>
                      <h2 class="text-result">Algo deu errado. Por gentileza, tente novamente mais tarde.</h2>
                    </div>
                  </v-card>
                </v-flex>
              </div>
              <v-btn v-if="result && !loading && reqFailed" round color="primary" dark @click="reset()">
                <i class="material-icons">mood</i>&nbsp;&nbsp;Simular Novamente
              </v-btn>
            </v-card>
          </v-flex>
          <v-btn fab dark class="btn-home" :title="titleCiclic" @click="goHome">
            <v-icon dark>home</v-icon>
          </v-btn>
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
  directives: {
    money: VMoney
  },
  data () {
    return {
      title: 'Simulador',
      subtitle: 'Na Ciclic seu sonho rende mais que na poupança',
      titleCiclic: 'Vem conhecer a Ciclic!',
      image: {
        url: require('../assets/img/ciclic.jpeg')
      },
      valid: true,
      user: {
        name: '',
        monthlyPayment: 'R$ 0,00',
        timeSimulator: 1,
        valueResult: ''
      },
      money: {
        decimal: ',',
        thousands: '.',
        prefix: 'R$ ',
        precision: 2,
        masked: false
      },
      nameRules: [
        v => !!v || 'Xiii, parece que você esqueceu de digitar o seu nome aqui :)',
        v => (v && v.length >= 3) || 'Psiu, o nome precisa ter pelo menos 3 letras, tá?! :)'
      ],
      monthlyRules: [
        v => (v !== 'R$ 0,00' && v !== 'R$ 0.00') || 'Psiu, a sua mensalidade tem que ser maior que 0 :)'
      ],
      result: false,
      loading: false,
      reqFailed: ''
    }
  },
  methods: {
    onlyLetter () {
      this.user.name = this.user.name.replace(/[^a-zA-Z.\s]/g, '')
    },
    submit () {
      const txJuros = '0.00517'

      if (this.$refs.form.validate()) {
        this.loading = true // Call screen loading when data is OK
        // ------- Preparing data for API ------- //
        let convertMonths = parseInt(this.user.timeSimulator) * 12
        let payment = this.user.monthlyPayment
        let rmvRS = payment.split('R$ ').join('') // Remove prefix add for mask money
        let rmvDot = rmvRS.split('.').join('') // Remove dot  add for mask money
        let rpcComma = rmvDot.replace(',', '.') // Change comma  for dot
        let newPayment = rpcComma

        let expr = `
          {
            "expr": [
              "${newPayment} * (((1 + ${txJuros}) ^ ${convertMonths} - 1) / ${txJuros})"
            ]
          }
        `
        axios({
          method: 'post',
          url: 'http://api.mathjs.org/v4/',
          data: expr
        }).then(response => {
          if (response.status === 200) { // Check status OK
            this.reqFailed = false
            this.user.valueResult = (parseFloat(response.data.result[0])).toFixed(2)
            let rpcDot = this.user.valueResult.replace('.', ',')
            let resultCurrency = rpcDot.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.') // Put mask money in value
            this.user.valueResult = resultCurrency
            this.loading = false // Remove screen loading
            this.result = true // Add second screen
            if (this.user.timeSimulator !== 0 && this.user.timeSimulator > 1) {
              this.user.timeSimulator = `${this.user.timeSimulator} anos`
            } else {
              if (this.user.timeSimulator === 1) {
                this.user.timeSimulator = `${this.user.timeSimulator} ano`
              }
            }
          } else {
            this.reqFailed = true
          }
        })
      }
    },
    reset () {
      this.user.name = ''
      this.user.monthlyPayment = 'R$ 0,00'
      this.user.timeSimulator = 1
      this.user.valueResult = ''
      this.result = false
    },
    goHome () {
      window.location.href = `https://www.ciclic.com.br/`
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
    display: block;
    padding: 0;
  }
  .subtitle-simulator{
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 30px;
    word-break: break-word;
  }
  .form {
    padding: 0 20px 20px 20px;
  }
  .v-text-field{
    color: #1976D2;
  }
  .label-start-time{
    float: left;
    font-size: 12px;
    color: rgba(0,0,0,.54);
  }
  .flex-time{
    width: 100%;
  }
  .slider{
    width: 75%;
    margin-top: 0;
  }
  .time-simulator{
    color: #324376;
    font-size: 16px;
    font-weight: 500;
    margin-left: 10px;
  }
  .v-btn{
    font-family: 'Montserrat', sans-serif;
    background-color: #324376 !important;
    margin-top: 20px;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
  .loading{
    width: 200px;
    height: 200px;
    margin: auto;
    display: flex;
    justify-content: center;
    align-content: center;
  }
  .loading .v-progress-circular{
    position: relative;
    top: 50%;
    transform: translateY(-50%);
  }
  .result{
    padding: 20px 30px 20px 30px;
  }
  .card-result{
    margin-top: -12px;
    background-color: rgb(245,245,245);
    height: auto;
  }
  .content-result{
    padding: 20px 20px 30px 20px;
  }
  .content-result h2{
    font-weight: 400;
    font-size: 17px;
    color: #324376;
    font-family: 'Montserrat', sans-serif;
  }
  .icon-result i{
    font-size: 50px;
    font-weight: bold;
    color: #324376;
  }
  .title-result{
    margin-bottom: 10px;
  }
  .title-result h2{
    font-size: 22px;
    text-transform: capitalize;
  }
  .text-result{
    word-break: break-word;
  }
  .btn-home{
    position: absolute;
    bottom: 20px;
    right: 40px;
    background-color: #E40066 !important;
  }
</style>
