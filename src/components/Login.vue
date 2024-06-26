<template>
    <div class="d-flex flex-column align-items-center container-fluid vh-100 p-0 m-0">
        <div class="d-flex align-items-center justify-content-center w-100 gap-2 py-2" style="background-color: #783582;">
            <h2 class="w-auto m-0 py-3">MeowAssist</h2>
            <div class="w-auto" style="max-width: 1.8em;">
                <img src="../Logo.png" style="max-width: 100%;">
            </div>
        </div>
        <div class="flex-grow-1 d-flex flex-column justify-content-center gap-4 w-50">
            <div>
                <label>Usuario</label>
                <input v-model="username" type="text" class="form-control" placeholder="Ingrese su usuario">
            </div>

            <div>
                <label>Contraseña</label>
                <div class="row w-100 g-0">
                    <div class="col p-0">
                        <input v-model="password" :type="passwordType.type" class="form-control" placeholder="Ingrese su contraseña">
                    </div>
                    <div class="col-auto p-0">
                        <button type="button" class="btn btn-secondary"
                            @click.prevent="isPassword = !isPassword">{{passwordType.text}}</button>
                    </div>
                </div>
            </div>

            <button type="button"
            class="btn btn-secondary"
            :disabled="username.trim() == '' || password == ''"
            @click.prevent="tryLoggin(username, password)">
                Ingresar
            </button>
        </div>
        <div class="row" style="height: 10%"></div>
    </div>
</template>

<script>
export default {
    props: ['tryLoggin'],
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
            username: '',
            password: '',
            isPassword: true
        }
    }
}
</script>