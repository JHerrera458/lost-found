<!-- eslint-disable vue/first-attribute-linebreak -->
<template>
  <v-container light>
    <div>
      <div class="spacer"></div>
      <h1>Administrar cuentas</h1>
      <v-dialog v-model="dialogDetails" width="500">
        <v-card color="grey lighten-4" light>
          <v-card-title class="light-blue darken-4" text-xs-center>
            Información de la cuenta
            <v-spacer></v-spacer>
            <v-icon @click="dialogDetails = false">mdi-close</v-icon>
          </v-card-title>
          <div style="margin-top: 25px"></div>
          <v-card-text>
            <v-img src="/defaultProfile.jpg" width="450px" height="350px">
            </v-img>
            <span class="title">ID: </span>
            {{ currentAccount.id }}
            <br />
            <span class="title">Nombre: </span>
            {{ currentAccount.name }}
            <br />
            <span class="title">Apellido: </span>
            {{ currentAccount.lastName }}
            <br />
            <span class="title">Número de celular: </span>
            {{ currentAccount.phoneNumber }}
            <br />
            <span class="title">Email: </span>
            {{ currentAccount.email }}
            <br />
            <span class="title">Rol: </span>
            {{ currentAccount.role }}
            <br />
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              class="light-blue darken-4"
              text
              @click="dialogDetails = false"
            >
              Cerrar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogEdit" max-width="500px">
        <v-card color="white" light>
          <v-card-title> Editar cuenta </v-card-title>
          <v-card-text>
            <v-form ref="">
              <v-text-field
                v-model="editInfo.name"
                :rules="[rules.required]"
                name="name"
                label="Nombre"
                outlined
                prepend-icon="mdi-account"
              />
              <v-text-field
                v-model="editInfo.lastName"
                :rules="[rules.required]"
                name="lastName"
                label="Apellido"
                outlined
                prepend-icon="mdi-account"
              />
              <v-text-field
                v-model="editInfo.phoneNumber"
                :rules="[rules.required]"
                name="number"
                label="Número de celular"
                outlined
                prepend-icon="mdi-phone"
              />
              <v-text-field
                v-model="editInfo.email"
                :rules="[rules.required, rules.email]"
                name="email"
                label="Correo electrónico"
                outlined
                prepend-icon="mdi-email"
              />
              <v-text-field
                v-model="editInfo.password"
                :append-icon="visiblePass ? 'mdi-eye' : 'mdi-eye-off'"
                :type="visiblePass ? 'text' : 'password'"
                :rules="[rules.required]"
                name="password"
                label="Contraseña"
                outlined
                prepend-icon="mdi-lock"
                @click:append="visiblePass = !visiblePass"
              />
              <v-col>
                <v-select
                  v-model="editInfo.role"
                  :items="roles"
                  label="Rol (1: Administrador || 0: Usuario)"
                  outlined
                  prepend-icon="mdi-badge-account"
                ></v-select>
              </v-col>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-btn color="#005b96" dark @click="updateAccount()">Editar</v-btn>
            <v-btn color="red" dark @click="dialogEdit = false">Cancelar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-row>
        <v-col
          v-for="account in accounts"
          :key="account.id"
          cols="3"
          align="center"
        >
          <v-card class="customCard">
            <v-card-title primary-title>
              {{ account.name }}
            </v-card-title>
            <v-card-text>
              <v-img src="/defaultProfile.jpg" width="350px" height="350px">
              </v-img>
            </v-card-text>
            <v-card-actions>
              <v-btn color="#005b96" dark @click="showDescription(account)"
                >Detalles</v-btn
              >
              <v-btn
                color="yellow darken-1"
                dark
                @click="loadAccountToUpdate(account)"
                >Editar</v-btn
              >
              <v-btn color="red" dark @click="deleteAccount(account)"
                >Eliminar</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>
<script>
export default {
  layout: 'defaultMaster',
  data() {
    return {
      roles: [0, 1],
      row: null,
      dialogEdit: false,
      dialogDetails: false,
      loading: false,
      visiblePass: false,
      accounts: [],
      currentAccount: {
        name: '',
        lastName: '',
        phoneNumber: '',
        email: '',
        password: '',
        role: 0,
        createDate: '',
        id: 0,
      },
      editInfo: {
        name: '',
        lastName: '',
        phoneNumber: '',
        email: '',
        password: '',
        role: 0,
        createDate: new Date(),
        id: 0,
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
    this.loadAccounts()
  },
  methods: {
    loadAccounts() {
      const url = 'http://localhost:3001/accounts'
      this.loading = true
      this.$axios
        .get(url)
        .then((response) => {
          this.accounts = response.data
        })
        .catch(() => {
          this.$swal.fire({
            title: 'Error!',
            text: 'Ha ocurrido un error cargando las cuentas.',
            icon: 'error',
          })
        })
        .finally(() => {
          this.loading = false
        })
    },
    showDescription(account) {
      this.dialogDetails = true
      this.currentAccount.id = account.id
      this.currentAccount.name = account.name
      this.currentAccount.lastName = account.lastName
      this.currentAccount.phoneNumber = account.phoneNumber
      this.currentAccount.email = account.email
      if (account.role === 0) {
        this.currentAccount.role = 'Usuario'
      } else {
        this.currentAccount.role = 'Administrador'
      }
    },
    deleteAccount(account) {
      this.$swal
        .fire({
          title: `¿Seguro de eliminar la cuenta: ${account.name}? con ID: ${account.id}`,
          text: 'El usuario se eliminará y no se puede recuperar',
          icon: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Si',
        })
        .then((result) => {
          if (result.isConfirmed) {
            const url = `http://localhost:3001/accounts/${account.id}`
            this.loading = true
            this.$axios
              .delete(url)
              .then((response) => {
                this.$swal.fire({
                  title: 'Cuenta eliminada',
                  text: `El usuario ${account.name} ID: ${account.id} se ha eliminado.`,
                  icon: 'success',
                })
                this.loadAccounts()
              })
              .catch(() => {
                this.$swal.fire({
                  title: 'Error!',
                  text: 'Ha ocurrido un error eliminando el usuario.',
                  icon: 'error',
                })
              })
              .finally(() => {
                this.loading = false
              })
          }
        })
    },
    loadAccountToUpdate(account) {
      this.dialogEdit = true
      this.editInfo.name = account.name
      this.editInfo.lastName = account.lastName
      this.editInfo.phoneNumber = account.phoneNumber
      this.editInfo.email = account.email
      this.editInfo.password = account.password
      this.editInfo.role = account.role
      this.editInfo.createDate = account.createDate
      this.editInfo.id = account.id
    },
    updateAccount() {
      const url = `http://localhost:3001/accounts/${this.editInfo.id}`
      this.loading = true
      this.$axios
        .put(url, this.editInfo)
        .then(() => {
          this.$swal.fire({
            icon: 'success',
            title: 'Cuenta actualiza',
            text: 'La cuenta fue actualizada exitosamente',
          })
          this.loadAccounts()
          this.dialog = false
        })
        .catch(() => {
          this.$swal.fire({
            icon: 'error',
            title: 'Hubo un error',
            text: 'No se pudo editar la cuenta',
          })
        })
        .finally(() => {
          this.loading = false
          this.dialogEdit = false
        })
    },
  },
}
</script>

<style>
.spacer {
  margin-bottom: 120px;
}
</style>
