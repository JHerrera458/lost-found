<template>
  <v-app light>
    <v-main light>
      <v-app-bar absolute color="#005b96" prominent>
        <v-app-bar-nav-icon @click="drawer = true" />
        <v-toolbar-title>
          <img src="/lostfound.png" alt="" class="image" />
        </v-toolbar-title>
        <v-spacer />
        <h1 class="title" />
        Bienvenido {{ fullname }}
      </v-app-bar>
      <v-navigation-drawer v-model="drawer" temporary absolute>
        <v-list>
          <v-list-item
            v-for="(item, i) in items"
            :key="i"
            :to="item.to"
            router
            exact
          >
            <v-list-item-action>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-action>
            <v-list-item-content>
              <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-navigation-drawer>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      drawer: false,
      fullname: '',
      items: [
        {
          icon: 'mdi-apps',
          title: 'Inicio',
          to: '/users/home',
        }
      ],
    }
  },
  beforeMount() {
    this.loadAccount()
  },
  methods: {
    loadAccount() {
      const accountStr = localStorage.getItem('account')
      if (accountStr) {
        const account = JSON.parse(accountStr)
        this.fullname = `${account.name}`
      } else {
        this.$router.push('/')
      }
    },
  },
}
</script>

<style>
.image {
  padding-top: 10px;
  height: 120px;
}
</style>
