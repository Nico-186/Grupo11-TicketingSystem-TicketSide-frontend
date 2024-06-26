<template>
    <div class="container-fluid h-100 v-100" style="padding-left:25%; padding-right: 25%">
        <div class="d-flex flex-column gap-4">
            <div class="d-flex justify-content-center mt-2 w-100">
                <h4 class="w-auto">{{ isCreate ? 'Crear Usuario' : 'Editar Usuario' }}</h4>
            </div>
            <div class="form-group w-100">
                <label>Nombre de usuario</label>
                <input ref="username" v-model="username" type="text" class="form-control" placeholder="Nombre de usuario">
            </div>
            <div class="form-group w-100">
                <label>Reestablecer contraseña</label>
                <input type="text" class="form-control" placeholder="Nueva contraseña" disabled :value="newPassword">
                <div class="d-flex justify-content-evenly pt-3">
                    <button type="button" class="btn btn-secondary" @click.prevent="passwordType(true)">Generar Nueva
                        contraseña</button>
                    <button type="button" class="btn btn-secondary" @click.prevent="passwordType(false)"
                        :disabled="newPassword == '' ? true : false">Eliminar Nueva contraseña</button>
                </div>
            </div>
            <div class="form-group w-100">
                <label>Rol del usuario</label>
                <ul class="list-group list-group-horizontal-lg">
                    <button v-for="(rol, index) in roleTypes" :id="index" type="button" class="list-group-item list-group-item-action" :style="activeListItem == index ? 'background-color: #3346C5; border-color: #3346C5;' : ''" :class="{ active: activeListItem == index}" @click.prevent="changeActiveListItem(index)">{{rol}}</button>
                </ul>
            </div>
            <div class="d-flex flex-column align-items-center gap-2">
                <button type="button"
                class="btn btn-success w-100"
                :disabled="username.trim() == '' || (newPassword == '' && oldPassword == '') || role == -1 || (username.trim() == user.nomusua && newPassword == '' && role == parseInt(user.rol))"
                @click.prevent="createUser(id, returnUser(), isCreate)">
                    {{ isCreate ? 'Crear usuario' : 'Guardar usuario' }}
                </button>
                <button type="button" class="btn btn-danger w-100" @click.prevent="deleteUser(id)" :style="{ visibility: !isCreate ? 'visible' : 'hidden' }">Eliminar usuario</button>
            </div>
        </div>
    </div>
</template>

<script>
import { SHA256 } from 'crypto-js';

export default {
    props: ['createUser', 'isCreate', 'user', 'deleteUser'],
    data() {
        return {
            id: '',
            username: '',
            newPassword: '',
            oldPassword: '',
            role: -1,
            activeListItem: -1,
            roleTypes: [ 'Usuario', 'Técnico', 'Administrador' ]
        }
    },
    methods: {
        returnUser() {
            if (this.newPassword == '') {
                return { username: this.username, password: this.oldPassword, role: this.role };
            } else {
                var encryptedPass = SHA256(this.newPassword + process.env.VUE_APP_SECRET).toString();
                return { username: this.username, password: encryptedPass, role: this.role };
            }
        },
        passwordType(generate) {
            if (generate) {
                this.newPassword = Math.random().toString(36).slice(-8);
            } else {
                this.newPassword = '';
            }
        },
        changeActiveListItem(index) {
            this.activeListItem = index;
            this.role = index;
        },
        showRoleOnList() {
            if (!this.isCreate) {
                this.activeListItem = this.role;
            }
        }
    },
    mounted() {
        if (!this.isCreate) {
            console.log(this.user);
            this.username = this.user.nomusua;
            this.id = this.user.ID_usuario;
            this.oldPassword = this.user.Contraseña;
            this.role = parseInt(this.user.rol);
            this.$refs.username.value = this.username;
            this.showRoleOnList()
        }
    }
}
</script>