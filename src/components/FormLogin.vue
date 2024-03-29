<template>
  <v-card class="mx-auto account-form-card">
    <v-card-title>{{
      loggedIn ? 'You already are logged in' : 'Log In'
    }}</v-card-title>
    <v-card-text>
      <v-form ref="form" v-model="valid" v-if="!loggedIn">
        <v-text-field
          v-model="email"
          :rules="emailRules"
          label="Email"
          required
          outlined
          @keyup.enter="login"
        ></v-text-field>
        <v-text-field
          :append-icon="showPass ? 'mdi-eye' : 'mdi-eye-off'"
          @click:append="showPass = !showPass"
          :type="showPass ? 'text' : 'password'"
          outlined
          :rules="passwordRules"
          hint="At least 8 characters"
          v-model="password"
          label="Password"
          required
          @keyup.enter="login"
        ></v-text-field>
        <p v-if="wrongPassword" class="red--text">
          Your username and/or password is incorrect.
        </p>
        <v-btn
          block
          color="success"
          @click="login"
          :disabled="!valid"
          :loading="loggingIn"
          >Log In</v-btn
        >
        <v-btn block class="mt-2" color="error" @click="reset">Reset</v-btn>
        <v-btn
          class="mt-2"
          block
          color="primary"
          @click="loginGoogle"
          :loading="loggingIn"
          ><v-icon left>mdi-google</v-icon>Login w/ Google</v-btn
        >
      </v-form>
    </v-card-text>
  </v-card>
</template>

<script>
import Firebase from '../firebase'
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'FormLogin',
  data() {
    return {
      valid: false,
      email: '',
      password: '',
      showPass: false,
      emailRules: [
        v => !!v || 'E-mail is required',
        v => /.+@.+\..+/.test(v) || 'E-mail must be valid'
      ],
      passwordRules: [
        v => !!v || 'Password is required',
        v => (v && v.length >= 8) || 'Password must be at least 8 characters'
      ],
      loggingIn: false,
      wrongPassword: false
    }
  },
  computed: {
    ...mapGetters(['loggedIn'])
  },
  methods: {
    loginGoogle() {
      this.loggingIn = true
      const provider = Firebase.googleProvider
      Firebase.auth
        .signInWithPopup(provider)
        .then(result => {
          this.loggingIn = false
          this.$router.push({ name: 'Home' })
        })
        .catch(error => {
          this.loggingIn = false
          const errorCode = error.code
          const errorMessage = error.message
          const email = error.email
          const credential = error.credential
          console.log(errorCode, errorMessage, email, credential)
        })
    },
    login() {
      if (this.valid) {
        this.loggingIn = true
        Firebase.auth
          .signInWithEmailAndPassword(this.email, this.password)
          .then(result => {
            this.loggingIn = false
            this.$router.push({ name: 'Home' })
          })
          .catch(error => {
            this.wrongPassword = true
            this.loggingIn = false
            var errorCode = error.code
            var errorMessage = error.message
            console.log(errorCode, errorMessage)
          })
      }
    },
    reset() {
      this.wrongPassword = false
      this.$refs.form.reset()
    }
  }
}
</script>

<style lang="scss" scoped>
.account-form-card {
  max-width: 320px;
  width: 100%;
}
</style>
