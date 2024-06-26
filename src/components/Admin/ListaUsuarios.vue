<template>
    <div class="container-fluid w-100 h-100 p-0" >
        <div class="border w-100 h-100" style="background-color: #171a1c;">
            <div class="d-flex flex-column h-100">
                <p class="w-100 mb-0 px-3 py-2 fs-5">Usuarios</p>

                <div class="flex-grow-1 overflow-auto w-100">
                    <div class="list-group h-100" style="max-height: 100%;">
                        <ul class="list-group">
                            <button v-for="(user, index) in users" :id="index" type="button" :style="activeListItem == index ? 'background-color: #3346C5; border-color: #3346C5;' : ''"
                                class="list-group-item list-group-item-action" :class="{ active: activeListItem == index && user.rol != ''}" @click.prevent="changeActiveListItem(index, user.rol)">
                                <div class="d-flex ms-2 me-auto align-items-center">
                                    <div class="fs-5 fw-bold">{{user.nomusua}}</div>
                                    <div style="white-space: pre;"> - {{roleTypes[user.rol]}} </div>
                                </div>
                            </button>
                        </ul>
                    </div>
                </div>

                <div class="d-flex w-100 justify-content-end p-3">
                    <button class="btn btn-secondary" type="button" href="" @click.prevent="editHandler(returnIsCreate())">{{ activeListItem == -1 ? 'Nuevo Usuario' : 'Editar Usuario' }}</button>
                </div>
            </div>
        </div>


    </div>
</template>

<script>
export default {
    props: ['users', 'editHandler'],
    data() {
        return {
            activeListItem: -1,
            roleTypes: [ 'Usuario', 'Técnico', 'Administrador' ]
        }
    },
    methods: {
        changeActiveListItem(index,rol) {
            if (this.activeListItem != index && rol != ''){
                this.activeListItem = index;
            } else {
                this.activeListItem = -1;
            };
        },
        returnIsCreate() {
            return this.activeListItem == -1 ? true : false;
        }
    }
}
</script>