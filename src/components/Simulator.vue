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
  directives: {
    money: VMoney
  },
  data () {
    return {
      title: 'Simulador',
      subtitle: 'Na Ciclic seu sonho rende mais que na poupança',
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
</style>
