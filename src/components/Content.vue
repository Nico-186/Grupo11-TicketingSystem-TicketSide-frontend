<template>
    <login v-if="!isLogged" :try-loggin="(username, password) => tryLogIn(username, password)"></login>
    <div v-if="isLogged" class="d-flex flex-column container-fluid vh-100 p-0" >

        <sidebar ref="sidebar" :active-page="activePage" :role="loggedUser.role" :sidebar-click="(page) => loadData(page)"></sidebar>
        <div id="content" class="container-fluid overflow-auto p-4 w-100 h-100">

            <ticketList v-if="activePage == 1"
            :ticket-list="ticketList"
            :status-list="listaEstatus"
            :priority-list="listaPrioridades"
            :user-list="userList"
            :open-selected-ticket="(ticketId) => showSelectedTicket(ticketId)" 
            ></ticketList>

            <ticketView v-if="activePage == 1.1"
            :logged-user="loggedUser"
            :ticket="selectedTicket"
            :status-list="listaEstatus"
            :priority-list="listaPrioridades"
            :user-list="userList"
            :assignable-users="assignableUsers"
            ></ticketView>

            <ticketForm v-if="activePage == 2"
            :priority-list="listaPrioridades"
            :id-log="loggedUser.id"
            ></ticketForm>

            <priority v-if="activePage == 3"
            :all-priorities="listaPrioridades"
            ></priority>

            <status v-if="activePage == 4"
            :all-status="listaEstatus"
            ></status>
        </div>
    </div>
    
</template>

<script>
import login from './Login.vue';
import sidebar from './Sidebar.vue';

import ticketList from './Ticket/List.vue';
import ticketView from './Ticket/View.vue';
import ticketForm from './Ticket/Form.vue';

import status from './Status.vue';
import priority from './Priority.vue';
import Status from './Status.vue';

export default {
    components: {
        login,
        sidebar,
        ticketList,
        ticketForm,
        ticketView,
        status,
        priority
    },
    data() {
        return {
            activePage: 0,
            isLogged: false,
            loggedUser: { id: -1, username: 'null', password: 'null', role: -1 },
            mounted: false,
            ticketList: [],
            listaEstatus: [],
            listaPrioridades: [],
            userList: [],
            assignableUsers: [],
            selectedTicket: {}
        }
    },
    mounted() {
        this.mounted = true
    },
    methods: {
        showSelectedTicket(ticketId) {
            this.selectedTicket = this.findTicketById(ticketId); 
            this.activePage = 1.1;
        },
        findTicketById(id) {
            return this.ticketList.find((obj) => obj.ID_ticket == id)
        },
        dataToLoggedUser(resData) {
            this.loggedUser = {
                id: resData.ID_usuario,
                username: resData.nomusua,
                password: resData.Contraseña,
                role: resData.rol
            } 
        },
        findAssignableUsers() {
            this.userList.forEach((user) => {
                if (Number(user.rol) > 0) {
                    this.assignableUsers.push(user);
                }
            });
        },
        async tryLogIn(user, pass) {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/logindata/?username=${user}&password=${pass}`).then(
                async (response) => {
                    if (JSON.stringify(response.data) == JSON.stringify([])) {
                        alert("Usuario o contraseña incorrectos");
                    } else {
                        this.dataToLoggedUser(response.data[0]);
                        this.isLogged = true;
                        this.loadData(1);
                    }
                }
            )
        },
        async loadData(page) {
            switch (page){
                case 1:
                    let wait1 = this.getTickets();
                    let wait2 = this.getStatus();
                    let wait3 = this.getPriority();
                    let wait4 = this.getusernames();
                    await wait1;
                    await wait2;
                    await wait3;
                    await wait4;
                    this.activePage = page;
                    break;
                case 2:
                    await this.getPriority();
                    this.activePage = page;
                    break;
                case 3:
                    await this.getPriority();
                    this.activePage = page;
                    break;
                case 4:
                    await this.getStatus();
                    this.activePage = page;
                    break;
            }
        },
        async getTickets() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/tickets`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.ticketList = response.data;
                    }
                }
            )
        },
        async getusernames() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/tickets/usernames`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.userList = response.data;
                        this.findAssignableUsers();
                    }
                }
            )
        },
        async getPriority() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaPrioridades = response.data;
                    }
                }
            )
        },
        async getStatus() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaEstatus = response.data;
                    }
                }
            )
        }
    }
}
</script>