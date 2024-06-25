<template>
    <nav class="navbar navbar-nav fixed-top" style="background-color: #AF1B47; position: relative; z-index: 2;">
        <div class="d-flex justify-content-start gap-4 container-fluid">
            <a class="m-0 h2 py-3" style="text-decoration: none;">MeowAssist Tickets</a>
            <a v-for="link in links(Number(loggedUser.role))" class="nav-link fs-6 py-2" :class="{ active: activePage >= link.page && activePage < link.page + 1 }" href="#" @click.prevent="sidebarClick(link.page)">{{link.text}}</a>
            <a class="nav-link fs-6 py-2" :href="link">Usuarios</a>
        </div>
    </nav>
</template>

<script>
export default {
    props: ['loggedUser', 'sidebarClick','activePage', 'changeToUser'],
    data () {
        return {
            link: process.env.VUE_APP_REDIRECT
        }
    },
    methods: {
        links(role) {
            let linkList = []

            linkList.push({text: 'Lista de tickets', page: 1})
            linkList.push({text: 'Crear nuevo ticket', page: 2})
            if (role == 2) {
                linkList.push({text: 'Gestionar prioridades', page: 3})
                linkList.push({text: 'Gestionar estatus', page: 4})
            }

            return linkList;
        }
    }
}
</script>