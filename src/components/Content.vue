<template>
    <login v-if="!isLogged" ref="loginComponent" :try-loggin="(username, password) => tryLogIn(username, password)"></login>
    <div class="container-fluid vh-100 p-0" :style=" isLogged ? '' : 'display: none;' ">

        <sidebar ref="sidebar" :role="loggedUser.role" :sidebar-click="(page) => activePage = page"></sidebar>

        <div v-if="isLogged" id="content" class="container-fluid overflow-auto" style="height: 90%;"
            :style="checkSidebar ? 'padding: 0% 0% 0% 17%;' : 'padding: 0%;'">

            <ticketList
            ref="adminUserList"
            v-if="activePage == 1"
            :tickets="ticketList"
            :status="listaEstatus"
            :prioridades="listaPrioridades"
            :user-list="userList"
            :open-selected-ticket="(ticketId) => showSelectedTicket(ticketId)" 
            ></ticketList>

            <ticketView
            v-if="activePage == 1.1"
            :ticket="selectedTicket"
            :logged-user="loggedUser"
            :status="listaEstatus"
            :prioridades="listaPrioridades"
            :user-list="userList"
            ></ticketView>

            <ticketForm
            v-if="activePage == 2"
            :prioridades="listaPrioridades"
            :id-log="loggedUser.id"
            ></ticketForm>
        </div>
    </div>
    
</template>

<script>
import login from './Login.vue';
import sidebar from './Sidebar.vue';

import ticketList from './Ticket/List.vue';
import ticketView from './Ticket/View.vue';
import ticketForm from './Ticket/Form.vue';

export default {
    components: {
        login,
        sidebar,
        ticketList,
        ticketForm,
        ticketView
    },
    computed: {
        checkSidebar() {
            if (this.mounted) {
                return this.$refs.sidebar.dashboard;
            } else {
                return false;
            }
        }
    },
    data() {
        return {
            activePage: 0,
            isLogged: false,
            loggedUser: { role: -1 },
            mounted: false,
            ticketList: [{
                ID_ticket: '',
                NAME: 'No existen tickets',
            }],
            createNewUser: false,
            listaEstatus: ['No existen estatus'],
            listaPrioridades: [],
            userList: [],
            selectedTicket: {}
        }
    },
    mounted() {
        this.mounted = true
    },
    methods: {
        tryLogIn: function (user, pass) {
            axios.get(`http://localhost:3000/logindata/?username=${user}&password=${pass}`).then(
                (response) => {
                    if (JSON.stringify(response.data) == JSON.stringify([])) {
                        alert("Usuario o contraseña incorrectos");
                    } else {
                        this.dataToLoggedUser(response.data[0]);
                        this.isLogged = true;
                        this.getTickets();
                        this.getStatus();
                        this.getPriority();
                        this.getusernames();
                    }
                }
            )
        },
        getTickets() {
            axios.get(`http://localhost:3000/tickets`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.ticketList = response.data;
                    }
                }
            )
        },
        getusernames() {
            axios.get(`http://localhost:3000/tickets/usernames`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.userList = response.data;
                    }
                }
            )
        },
        getPriority() {
            axios.get(`http://localhost:3000/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaPrioridades = response.data;
                    }
                }
            )
        },
        getStatus() {
            axios.get(`http://localhost:3000/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaEstatus = response.data;
                    }
                }
            )
        },
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
        }
    }
}
</script>