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
                    <v-btn class="green white--text" :disabled="editing"
                        @click="createObject"><v-icon>mdi-upload</v-icon></v-btn>
                    <v-btn class="yellow darken-1 white--text" :disabled="!editing"
                        @click="updateObject"><v-icon>mdi-pencil</v-icon></v-btn>
                    <v-btn class="red white--text" :disabled="!editing"><v-icon>mdi-delete</v-icon></v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-row>
            <v-col cols="12">
                <v-btn color="light-blue darken-4">
                    <h2 @click="openDialog()">Registrar un nuevo objeto perdido</h2>
                </v-btn>
            </v-col>
        </v-row>
        <v-divider class="divider"></v-divider>
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
                        <v-btn color="light-blue darken-4" width="100%" class="title" @click="loadObjectToUpdate(object)">
                            Ver
                            detalles</v-btn>
                    </v-card-actions>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
export default {
    layout: 'defaultMaster',
    data() {
        return {
            editing: false,
            loading: false,
            dialogCreateObject: false,
            categories: [],
            objects: [],
            registerObject: {
                name: "",
                description: "",
                foundPlace: "",
                category: "",
                createdBy: 0,
                createDate: new Date(),
                isFound: false,
            },
            updateId: null
        }
    },
    beforeMount() {
        this.loadSession()
        this.loadCategories()
        this.loadObjects()
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
        openDialog() {
            this.clearFields()
            this.editing = false
            this.dialogCreateObject = true
        },
        clearFields() {
            this.updateId = ""
            this.registerObject.name = ""
            this.registerObject.description = ""
            this.registerObject.foundPlace = ""
            this.registerObject.category = ""
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
                this.loadObjects()
                this.dialogCreateObject = false
                this.clearFields()
            }).catch(() => {
                this.$swal.fire({
                    icon: 'error',
                    title: 'Ocurrió un error creando el objeto perdido',
                })
            }).finally(() => {
                this.loading = false
            })
        },
        loadObjects() {
            const url = `${process.env.URL_DEV}/lostObjects`
            this.$axios.get(url).then(response => {
                this.objects = response.data
            }).catch(() => {
            })
        },
        loadObjectToUpdate(object) {
            this.editing = true
            this.dialogCreateObject = true
            this.updateId = object.id
            this.registerObject.name = object.name
            this.registerObject.description = object.description
            this.registerObject.foundPlace = object.foundPlace
            this.registerObject.category = object.category
            this.registerObject.createdBy = object.createdBy
            this.registerObject.createDate = object.createDate
            this.registerObject.isFound = object.isFound
        },
        updateObject() {
            const url = `${process.env.URL_DEV}/lostObjects/${this.updateId}`
            this.loading = true
            this.$axios.put(url, this.registerObject).then(() => {
                this.$swal.fire({
                    icon: 'success',
                    title: 'Cuenta actualiza',
                    text: 'La cuenta fue actualizada exitosamente',
                })
                this.loadObjects()
                this.dialogCreateObject = false
                this.editing = false
            }).catch(() => {
                this.$swal.fire({
                    icon: 'error',
                    title: 'Hubo un error',
                    text: 'No se pudo editar la cuenta',
                })
            }).finally(() => {
                this.loading = false
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
    font-size: 100%;
}

.cardText {
    color: #005b96;
    font-size: 100%;
}
</style>