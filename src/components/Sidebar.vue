<template>
    <nav class="navbar navbar-nav fixed-top" style="background-color: #C21E4F; position: relative; z-index: 2;">
        <div class="d-flex justify-content-start gap-4 container-fluid">
            <a class="m-0 h2 py-3" style="text-decoration: none;">MeowAssist Tickets</a>
            <a v-for="link in links(role)" class="nav-link fs-6 py-2" :class="{ active: activePage == link.page }" href="#" @click.prevent="sidebarClick(link.page)">{{link.text}}</a>
        </div>
    </nav>
</template>

<script>
export default {
    props: ['role', 'sidebarClick','activePage'],
    data(){
        return {
          dashboard: false,
        }
    },
    methods: {
        showDashboard(){
            this.dashboard = !this.dashboard
        },
        links(role) {
            let linkList = []

            linkList.push({text: 'Lista de tickets', page: 1})
            linkList.push({text: 'Crear nuevo ticket', page: 2})
            linkList.push({text: 'Gestionar prioridades', page: 3})
            linkList.push({text: 'Gestionar estatus', page: 4})
            linkList.push({text: 'Usuarios', page:5})

            return linkList;
        }
    }
}
</script>