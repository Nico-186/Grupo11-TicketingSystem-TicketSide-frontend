<template>
    <div class="form-group w-100">
        <label>{{text}}</label>
        <div class="row w-100 g-0">
            <div class="col p-0">
                <input v-model="password" :type="passwordType.type" class="form-control" :placeholder="placeholderText">
            </div>
            <div class="col-auto p-0">
                <button type="button" class="btn btn-secondary"
                    @click.prevent="isPassword = !isPassword">{{passwordType.text}}</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ['text', 'placeholderText'],
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
            password: '',
            isPassword: true
        }
    },
}
</script>