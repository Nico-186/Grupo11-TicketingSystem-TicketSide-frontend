<template>
    <div class="container-fluid vh-100">
        <div class="row align-items-center justify-content-center" style="height: 10%; background-color: #426A39;">
            <h2 class="w-auto m-0">MeowAssist</h2>
        </div>
        <div class="row align-items-center justify-content-center" style="height: 80%;">
            <div class="col-lg-4 col-sm-11">
                <div class="d-grid gap-3">
                    <div class="form-group">
                        <label>Usuario</label>
                        <input v-model="username" type="text" class="form-control" placeholder="Ingrese su usuario">
                    </div>
                    <password ref="passInput" text="Contraseña" placeholder-text="Ingrese su contraseña"></password>
                    <button type="button" class="btn btn-secondary"
                        @click.prevent="setValues(); tryLoggin(username, password)">Ingresar</button>
                </div>
            </div>
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