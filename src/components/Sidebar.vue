<template>
    <nav class="navbar navbar-collapse fixed-top p-0" style="height: 10%; background-color: #C21E4F; position: relative; z-index: 2;">
        <div class="container-fluid">
            <button class="navbar-toggler" type="button" @click.prevent="showDashboard()">
                <span class="navbar-toggler-icon"></span>
            </button>
            <h2 class="w-auto m-0">MeowAssist Tickets</h2>
            <span class="navbar-toggler-icon px-3" style="visibility: hidden;"></span>
        </div>
    </nav>
    <div class="offcanvas offcanvas-start h-100" style="z-index: 1; width: 17%;" :class="dashboard ? 'show' : ''"
        tabindex="-1" :style="{ visibility: dashboard ? 'visible' : 'hidden' }">
        <div class="offcanvas-body py-0">
            <div class="w-100" style="height: 10%;"></div>
            <a v-for="link in links(role)" class="nav-link active fs-6 py-2" href="" @click.prevent="sidebarClick(link.page)">{{link.text}}</a>
        </div>
    </div>
</template>

<script>
export default {
    props: ['role', 'sidebarClick'],
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

            return linkList;
        }
    }
}
</script>