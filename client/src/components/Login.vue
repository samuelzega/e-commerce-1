<template>
  <v-layout row justify-center dark>
    <v-dialog v-model="show" persistent max-width="400px">
      <v-card>
        <v-card-title primary-title align-center class="headline lighten-2 justify-center">
          Login
        </v-card-title>
        <v-card-text>
          <v-container grid-md>
            <v-layout row wrap>
              <v-flex xs12>
                <v-form @submit.prevent="login()" id="login-form" xs12>
                    <v-text-field v-model="email" label="Email*" type="email" required
                    :rules="emailRules"></v-text-field>
                    <v-text-field  v-model="password" label="Password*" type="password" required
                    :rules="passwordRule"></v-text-field>
                    <v-btn type="submit" color="success" form="login-form" block>Login</v-btn>
                </v-form>
              </v-flex>
            </v-layout>
          </v-container>
          <small>*required field</small>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" flat @click="show = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import { mapMutations } from 'vuex'
import myaxios from '@/api/axios'
import axiosErrorHandler from '@/api/axiosErrorHandler'
export default {

  data () {
    return {
      email: '',
      password: '',
      emailRules: [
        (v) => v.length > 0 || 'E-mail is required',
        (v) => /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'E-mail must be valid'
      ],
      passwordRule: [
        (v) => !!v || 'Password is required'
      ]
    }
  },
  props: {
    value: Boolean
  },
  computed: {
    show: {
      get () {
        return this.value
      },
      set (value) {
        this.$emit('input', value)
      }
    }
  },
  methods: {
    ...mapMutations(['showSnackbar']),
    login () {
      let data = {
        email: this.email,
        password: this.password
      }
      myaxios({
        method: 'post',
        url: '/users/login',
        data
      })
        .then(({ data }) => {
          localStorage.token = data.token
          if(data.role === "customer"){
            this.$store.dispatch('getUserCart')
            this.$store.dispatch('getUserOrders')
          }
          this.$store.commit('setLogin', true)
          this.$store.commit('setUser', data)
          this.show = false
          this.showSnackbar({ text: 'Login Success' })
        })
        .catch(error => {
          axiosErrorHandler(error)
        })
    }
  }
}
</script>
