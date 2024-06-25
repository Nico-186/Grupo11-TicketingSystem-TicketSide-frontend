<template>
    <login v-if="!isLogged" :try-loggin="(username, password) => tryLogIn(username, password)"></login>
    <div v-if="isLogged" class="d-flex flex-column container-fluid vh-100 p-0" >

        <sidebar ref="sidebar" :active-page="activePage" :logged-user="loggedUser" :sidebar-click="(page) => loadData(page)"></sidebar>
        <div id="content" class="container-fluid overflow-auto p-4 w-100 h-100">

            <ticketList v-if="activePage == 1"
            :ticket-list="ticketList"
            :status-list="listaEstatus"
            :priority-list="listaPrioridades"
            :user-list="userList"
            :open-selected-ticket="(ticketId) => showSelectedTicket(ticketId)"
            :logged-user="loggedUser"
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
            :all-tickets="ticketList"
            ></priority>

            <status v-if="activePage == 4"
            :all-status="listaEstatus"
            :all-tickets="ticketList"
            ></status>
        </div>
    </div>
    
</template>

<script>
import cookies from 'vue-cookies';
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
            ticketList: [],
            listaEstatus: [],
            listaPrioridades: [],
            userList: [],
            assignableUsers: [],
            selectedTicket: {}
        }
    },
    methods: {
        isFirstLogin(){
            if (this.loggedUser.isFirst == 1) {
                cookies.set('loggedUser', { ID_usuario: this.loggedUser.id, justCreated: { data: [this.loggedUser.isFirst] } }, '30min');
                window.location.href = process.env.VUE_APP_REDIRECT;
            }
        },
        showSelectedTicket(ticketId) {
            this.selectedTicket = this.findTicketById(ticketId); 
            this.activePage = 1.1;
        },
        findTicketById(id) {
            return this.ticketList.find((obj) => obj.ID_ticket == id)
        },
        dataToLoggedUser(resData) {
            console.log(resData)
            this.loggedUser = {
                id: resData.ID_usuario,
                username: resData.nomusua,
                password: resData.Contraseña,
                role: resData.rol,
                isFirst: resData.justCreated.data[0]
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
                    if (JSON.stringify(response.data[0]) == JSON.stringify([])) {
                        alert("Usuario o contraseña incorrectos");
                    } else {
                        this.dataToLoggedUser(response.data[0][0]);
                        this.isLogged = true;
                        this.isFirstLogin();
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
                        this.ticketList = response.data[0];
                    }
                }
            )
        },
        async getusernames() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/tickets/usernames`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.userList = response.data[0];
                        this.findAssignableUsers();
                    }
                }
            )
        },
        async getPriority() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaPrioridades = response.data[0];
                    }
                }
            )
        },
        async getStatus() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL}/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaEstatus = response.data[0];
                    }
                }
            )
        }
    }
}
</script>