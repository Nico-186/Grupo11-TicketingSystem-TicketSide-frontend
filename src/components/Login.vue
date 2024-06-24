<template>
    <div class="d-flex flex-column align-items-center container-fluid vh-100 p-0 m-0">
        <div class="row align-items-center justify-content-center w-100" style="height: 10%; background-color: #426A39;">
            <h2 class="w-auto m-0">MeowAssist</h2>
        </div>
        <div class="flex-grow-1 d-flex flex-column justify-content-center gap-4 w-50">
            <div>
                <label>Usuario</label>
                <input v-model="username" type="text" class="form-control" placeholder="Ingrese su usuario">
            </div>
            <password ref="passInput" text="Contraseña" placeholder-text="Ingrese su contraseña"></password>
            <button type="button" class="btn btn-secondary"
                @click.prevent="setValues(); tryLoggin(username, password)">Ingresar</button>
        </div>
        <div class="row" style="height: 10%"></div>
    </div>
</template>

<script>
import password from './Utils/Password.vue';

export default {
    props: ['tryLoggin'],
    components: {
        password
    },
    methods: {
        setValues() {
            this.password = this.$refs.passInput.password;
        }
    }
}
</script>