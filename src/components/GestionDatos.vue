<template>
    <div class="container-fluid h-100 v-100" style="padding-left:25%; padding-right: 25%">
        <div class="d-flex flex-column align-items-center gap-4">
            <div class="d-flex justify-content-center mt-2 w-100">
                <h4 class="w-auto">{{ user.isFirst != 1 ? 'Editar datos' : 'Debe cambiar su contraseña' }}</h4>
            </div>
            <div v-if="user.isFirst != 1" class="form-group w-100">
                <label>Nombre de usuario</label>
                <input v-model="newUsername" ref="usrText" type="text" class="form-control" placeholder="Nombre de usuario">
            </div>
            <div class="form-group w-100">
                <label v-if="user.isFirst != 1" class="pb-2">Cambiar contraseña</label>
                <div class="form-group w-100">
                    <label>Ingrese nueva contraseña</label>
                    <div class="row w-100 g-0">
                        <div class="col p-0">
                            <input v-model="newPassword" :type="passwordType.type" class="form-control" placeholder="Nueva contraseña">
                        </div>
                        <div class="col-auto p-0">
                            <button type="button" class="btn btn-secondary"
                                @click.prevent="isPassword = !isPassword">{{passwordType.text}}</button>
                        </div>
                    </div>
                    <label class="d-flex flex-column w-100 border border-dark-subtle px-3 py-1 rounded-2" style="opacity: 0.6;">Debe contener:
                        <label class="ps-4">● Al menos 8 caracteres</label>
                        <label class="ps-4">● Al menos 1 numeros</label>
                        <label class="ps-4">● Letras mayusculas y minusculas</label>
                    </label>
                </div>
                <button v-if="user.isFirst != 1" type="button"
                class="btn btn-success my-4 w-100"
                :disabled="!validatePass(newUsername,newPassword,user.username)"
                @click.prevent="updateUser(returnUser())">Guardar cambios</button>
                <button v-if="user.isFirst == 1" type="button"
                class="btn btn-success my-4 w-100"
                :disabled="newPassword == '' || newPassword.length < 8 || !(new RegExp('[A-Z]').test(newPassword) && new RegExp('[a-z]').test(newPassword) && new RegExp('[0-9]').test(newPassword))"
                @click.prevent="updateUser(returnUser()),updateFirst()">Cambiar contraseña</button>
            </div>
        </div>
    </div>
</template>

<script>
import { SHA256 } from 'crypto-js';

export default {
    props: ['user','updateUser'],
    computed: {
        passwordType() {
            if (this.isPassword) {
                return {
                    type: 'password',
                    text: 'Mostrar'
                }
            } else {
                return {
                    type: 'text',
                    text: 'Ocultar'
                }
            }
        },
    },
    data() {
        return {
            newUsername: '',
            newPassword: '',
            password: '',
            isPassword: true
        }
    },
    mounted() {
        this.newUsername = this.user.username;
    },
    methods: {
        validatePass(newUsr, newPass, oldUsr){
            if(newUsr == ''){
                return false;
            } else if (newUsr.trim() != oldUsr && (newPass == '' || (newPass.length >= 8 && new RegExp('[A-Z]').test(newPass) && new RegExp('[a-z]').test(newPass) && new RegExp('[0-9]').test(newPass)))){
                return true;
            } else if (newUsr.trim() == oldUsr && (newPass.length >= 8 && new RegExp('[A-Z]').test(newPass) && new RegExp('[a-z]').test(newPass) && new RegExp('[0-9]').test(newPass))){
                return true;
            } else {
                return false;
            }
        },
        returnUser() {
            if (this.newPassword == '') {
                return { id: this.user.id, username: this.newUsername.trim(), password: this.user.password, role: Number(this.user.role) };
            } else {
                var encryptedPass = SHA256(this.newPassword + process.env.VUE_APP_SECRET).toString();
                return { id: this.user.id, username: this.newUsername.trim(), password: encryptedPass, role: Number(this.user.role) };
            }
        },
        async updateFirst(){
            await axios.put(`${process.env.VUE_APP_BACKENDURL_USER}/admin/allusers/firstloggin/?id=${this.user.id}`).then(
                (response) => {
                    window.location.href = location.reload();
                }).catch((error) =>{
                    return alert(error);
                })
        }
    }
}
</script>