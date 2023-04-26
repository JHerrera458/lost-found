<!-- eslint-disable vue/first-attribute-linebreak -->
<template>
    <v-container>
        <div class="spacer"></div>
        <h1>Gestionar objetos perdidos</h1>
        <v-dialog v-model="dialogCreateObject" width="500">
            <v-card color="white" light>
                <v-card-title class="light-blue darken-4">
                    <h2 class="title"> Registrar un objeto perdido</h2>
                </v-card-title>
                <v-card-text>
                    <v-divider class="divider"></v-divider>
                    <v-text-field v-model="registerObject.name" label="Nombre del objeto" outlined></v-text-field>
                    <v-textarea v-model="registerObject.description" label="Descripción del objeto"
                        hint="Ingrese una descripción del objeto" rows="3" row-height="25" outlined>
                    </v-textarea>
                    <v-text-field v-model="registerObject.foundPlace" label="Lugar dónde se encontró"
                        outlined></v-text-field>
                    <v-combobox v-model="registerObject.category" :items="categories" label="Categoria del objeto"
                        item-text="name" item-value="id" outlined light :menu-props="{ light: true }"></v-comboBox>
                    <v-divider class="divider"></v-divider>
                </v-card-text>
                <v-card-actions>
                    <v-btn class="green white--text" @click="createObject"><v-icon>mdi-upload</v-icon></v-btn>
                    <v-btn class="yellow darken-1 white--text"><v-icon>mdi-pencil</v-icon></v-btn>
                    <v-btn class="red white--text"><v-icon>mdi-delete</v-icon></v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-row>
            <v-col cols="12">
                <v-btn color="light-blue darken-4">
                    <h2 @click="dialogCreateObject = true">Registrar un nuevo objeto perdido</h2>
                </v-btn>
            </v-col>
        </v-row>
        <v-divider class="divider"></v-divider>
    </v-container>
</template>

<script>
export default {
    layout: 'defaultMaster',
    data() {
        return {
            loading: false,
            dialogCreateObject: false,
            categories: [],
            registerObject: {
                name: "",
                description: "",
                foundPlace: "",
                category: "",
                createdBy: 0,
                createDate: new Date(),
            }
        }
    },
    beforeMount() {
        this.loadCategories()
        this.loadSession()
    },
    methods: {
        loadSession() {
            const accountStr = localStorage.getItem('account')
            if (accountStr) {
                const account = JSON.parse(accountStr)
                this.registerObject.createdBy = account.id
            } else {
                this.$router.push('/')
            }
        },
        loadCategories() {
            const url = `${process.env.URL_DEV}/objectCategories`
            this.$axios.get(url).then(response => {
                this.categories = response.data
            }).catch(() => {
            })
        },
        createObject() {
            this.loading = true
            const url = `${process.env.URL_DEV}/lostObjects`
            this.$axios.post(url, this.registerObject).then(() => {
                this.$swal.fire('Objeto perdido creado!', 'Exitosamente!', 'success')
            }).catch(() => {
                this.$swal.fire({
                    icon: 'error',
                    title: 'Ocurrió un error creando el objeto perdido',
                })
            }).finally(() => {
                this.loading = false
                this.dialogCreateObject = false
            })
        }
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
}
</style>