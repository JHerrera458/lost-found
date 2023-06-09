<!-- eslint-disable vue/first-attribute-linebreak -->
<template>
  <div class="content">
    <v-row align="center">
      <v-col cols="6" align="right">
        <v-img src="/lostfound.png" max-width="400px" max-height="400px" />
      </v-col>
      <v-col cols="6" align="left">
        <v-card class="customCard" color="white" light>
          <v-card-title primary-title>
            <h1>Inicia sesión</h1>
          </v-card-title>
          <v-card-text>
            <v-form ref="formLoginAccount" v-model="formLoginAccount" />
            <v-text-field v-model="email" name="email" label="Correo electrónico" outlined prepend-icon="mdi-email" />
            <v-text-field v-model="password" :append-icon="visiblePass ? 'mdi-eye' : 'mdi-eye-off'"
              :type="visiblePass ? 'text' : 'password'" name="password" label="Contraseña" outlined
              prepend-icon="mdi-lock" @click:append="visiblePass = !visiblePass" />
          </v-card-text>
          <v-card-actions>
            <v-btn dark class="customBtn" color="#03396c" :loading="loading" @click="loginAccount()">
              Iniciar sesión
            </v-btn>
            <v-btn dark class="customBtn" color="#005b96" @click="dialog = !dialog">
              Crear una cuenta
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
    <v-dialog v-model="dialog" max-width="500">
      <v-card color="white" light>
        <v-card-title> Crea una cuenta nueva </v-card-title>
        <v-card-text>
          <v-form ref="formCreateAccount" v-model="formCreateAccount">
            <v-text-field v-model="registerInfo.name" :rules="[rules.required]" name="name" label="Nombre" outlined
              prepend-icon="mdi-account" />
            <v-text-field v-model="registerInfo.lastName" :rules="[rules.required]" name="lastName" label="Apellido"
              outlined prepend-icon="mdi-account" />
            <v-text-field v-model="registerInfo.phoneNumber" :rules="[rules.required]" name="number"
              label="Número de celular" outlined prepend-icon="mdi-phone" />
            <v-text-field v-model="registerInfo.email" :rules="[rules.required, rules.email]" name="email"
              label="Correo electrónico" outlined prepend-icon="mdi-email" />
            <v-text-field v-model="registerInfo.password" :append-icon="visiblePass ? 'mdi-eye' : 'mdi-eye-off'"
              :type="visiblePass ? 'text' : 'password'" :rules="[rules.required]" name="password" label="Contraseña"
              outlined prepend-icon="mdi-lock" @click:append="visiblePass = !visiblePass" />
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn dark color="#005b96" :loading="loading" @click="createAccount()">
            Crear una cuenta
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  layout: 'blank',
  data() {
    return {
      accountExist: false,
      loading: false,
      formCreateAccount: null,
      formLoginAccount: null,
      visiblePass: false,
      dialog: false,
      email: '',
      password: '',
      registerInfo: {
        name: '',
        lastName: '',
        phoneNumber: '',
        email: '',
        password: '',
        role: 0,
        createDate: new Date(),
      },
      rules: {
        required: (value) => !!value || 'Este campo es obligatorio',
        email: (value) => {
          const pattern =
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
          return pattern.test(value) || 'Ingresa un email válido'
        },
      },
    }
  },
  beforeMount() {
    this.clearStorage()
  },
  methods: {
    clearStorage() {
      localStorage.clear()
    },
    async validateAccountExistance() {
      const url = `${process.env.URL_DEV}/users`
      this.loading = true
      await this.$axios
        .get(url)
        .then((response) => {
          const data = response.data.info
          const found = data.find(
            (account) => account.email === this.registerInfo.email
          )
          if (found) {
            this.accountExist = true
          } else {
            this.accountExist = false
          }
        })
        .catch(() => {
          this.accountExist = null
        })
        .finally(() => {
          this.loading = false
        })
    },
    async createAccount() {
      const refFormCreateAccount = this.$refs.formCreateAccount
      if (refFormCreateAccount) {
        const formIsValid = refFormCreateAccount.validate()
        if (formIsValid) {
          await this.validateAccountExistance()
          if (this.accountExist) {
            this.$swal.fire({
              icon: 'warning',
              title: 'Correo ya registrado',
            })
          } else {
            this.loading = true
            const url = `${process.env.URL_DEV}/users`
            this.$axios
              .post(url, this.registerInfo)
              .then(() => {
                this.$swal.fire('Cuenta creada!', 'Exitosamente!', 'success')
              })
              .catch(() => {
                this.$swal.fire({
                  icon: 'error',
                  title: 'Ocurrió un error creando la cuenta',
                })
              })
              .finally(() => {
                this.loading = false
                this.dialog = false
              })
          }
        } else {
          this.$swal.fire({
            icon: 'warning',
            title: 'Faltan campos en el formulario.',
          })
        }
      }
    },
    loginAccount() {
      const url = `${process.env.URL_DEV}/users`
      this.loading = true
      this.$axios
        .get(url)
        .then((response) => {
          const data = response.data.info
          const found = data.find(
            (account) =>
              account.email === this.email && account.password === this.password
          )
          if (found) {
            const id = found._id
            const name = found.name
            const email = found.email
            const phoneNumber = found.phoneNumber
            const role = found.role
            this.$swal.fire({
              title: 'Iniciaste sesión!',
              text: 'Bienvenido.',
              icon: 'success',
            })
            if (found.role === 1) {
              localStorage.setItem(
                'account',
                JSON.stringify({ id, name, email, phoneNumber, role })
              )
              this.$router.push('/masters/home')
            } else {
              localStorage.setItem(
                'account',
                JSON.stringify({ id, name, email, phoneNumber, role })
              )
              this.$router.push('/users/home')
            }
          } else {
            this.$swal.fire({
              title: 'Error!',
              text: 'Usuario y/o contraseña incorrectos.',
              icon: 'error',
            })
          }
        })
        .catch(() => {
          this.$swal.fire({
            title: 'Error!',
            text: 'Ha ocurrido un error.',
            icon: 'error',
          })
        })
        .finally(() => {
          this.loading = false
        })
    },
  },
}
</script>

<style>
.content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(to bottom, #baeeff, #8393ff);
}

.customBtn {
  width: 200px;
}

.customCard {
  max-width: 425px;
}
</style>
