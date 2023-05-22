<template>
  <v-container light>
    <div class="spacer"></div>
    <h1>Lista de objetos perdidos actualmente</h1>
    <v-row>
      <v-col v-for="object in objects" :key="object.id" cols="2" align="center">
        <v-card color="white">
          <v-card-title primary-title>
            <h2 class="cardText">{{ object.name }}</h2>
          </v-card-title>
          <v-card-text>
            <v-img src="/object.png" width="200px" height="200px">
            </v-img>
          </v-card-text>
          <v-card-actions>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
export default {
  layout: "defaultUser",
  data() {
    return {
      objects: [],
    }
  },
  beforeMount() {
    this.loadObjects()
  },
  methods: {
    loadObjects() {
      const url = `${process.env.URL_DEV}/lostObjects`
      this.loading = true
      this.$axios.get(url).then(response => {
        this.objects = response.data
      }).catch(() => {
        this.$swal.fire({
          icon: 'error',
          title: 'Ocurrió un error cargando los objetos perdidos',
        })
      }).finally(() => {
        this.loading = false
      })
    },
  }
}
</script>
<style>
.spacer {
  margin-bottom: 120px;
}
.divider {
  margin-top: 20px;
  margin-bottom: 20px;
}
.title {
  color: white;
  font-size: 100%;
}
.cardText {
  color: #005b96;
  font-size: 100%;
}
</style>