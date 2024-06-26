<template>
    <nav class="navbar navbar-nav fixed-top" style="position: relative; z-index: 2;" :style="isTicket ? 'background-color: #AF1B47;' : 'background-color: #3346C5;'">
        <div class="d-flex align-items-center justify-content-start gap-4 w-100 px-3">
            <div class="d-flex align-items-center gap-2">
                <a class="m-0 h2 py-3" style="text-decoration: none;">MeowAssist {{ isTicket ? 'Tickets' : 'Usuarios'}}</a>
                <div class="w-auto" style="max-width: 1.8em;">
                    <img src="../Logo.png" style="max-width: 100%;">
                </div>
            </div>
            <a v-for="link in links(Number(loggedUser.role),isTicket)" class="nav-link fs-6 py-2" :class="{ active: activePage >= link.page && activePage < link.page + 1 }" href="#" @click.prevent="sidebarClick(link.page)">{{link.text}}</a>
            <a v-if="loggedUser.isFirst != 1" href="#" class="nav-link fs-6 py-2" @click="changeFront()">{{ !isTicket ? 'Tickets' : 'Usuarios'}}</a>
            <div class="flex-grow-1"></div>
            <a v-if="loggedUser.isFirst != 1" href="/" class="nav-link fs-6 py-2">Cerrar sesión</a>
        </div>
    </nav>
</template>

<script>
export default {
    props: ['loggedUser','isTicket', 'sidebarClick','activePage', 'changeToUser','changeFront'],
    methods: {
        links(role,isTicket) {
            let linkList = []

            if (isTicket){
                linkList.push({text: 'Lista de tickets', page: 1})
                linkList.push({text: 'Crear nuevo ticket', page: 2})
                if (role == 2) {
                    linkList.push({text: 'Gestionar prioridades', page: 3})
                    linkList.push({text: 'Gestionar estatus', page: 4})
                }
            } else {
                if (this.loggedUser.isFirst != 1){
                    linkList.push({text: 'Cuenta', page: 5})
                    if (role == 2) {
                        linkList.push({text: 'Administrar usuarios', page: 6})
                    }
                }
            }

            return linkList;
        }
    }
}
</script>