<template>
    <login v-if="!isLogged" :try-loggin="(username, password) => tryLogIn(username, password)"></login>
    <div v-if="isLogged" class="d-flex flex-column container-fluid vh-100 p-0">

        <sidebar ref="sidebar"
        :is-ticket="isTicket"
        :active-page="activePage"
        :logged-user="loggedUser"
        :change-front="() => changeFront()"
        :sidebar-click="(page) => loadData(page)"></sidebar>
        <div id="content" class="container-fluid overflow-auto p-4 w-100 h-100">

            <ticketList v-if="activePage == 1"
            :ticket-list="ticketList"
            :status-list="listaEstatus"
            :priority-list="listaPrioridades"
            :user-list="userList"
            :assignable-users="assignableUsers"
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

            <gestionDatos 
            v-if="activePage == 5" 
            :user="loggedUser"
            :update-user="(user) => createUser(user.id,user,false,5)"
            ></gestionDatos>

            <listaUsuariosAdmin 
            ref="adminUserList"
            v-if="activePage == 6"
            :users="usersList"
            :edit-handler="(isCreate) => {createNewUser = isCreate; activePage = 6.1}"
            ></listaUsuariosAdmin> 
                
            <gestionUsuario 
            v-if="activePage == 6.1" 
            :create-user="(id,user,isCreate) => createUser(id,user,isCreate,6)"
            :is-create="createNewUser"
            :user="getSelectedListIndex()"
            :delete-user="(id) => deleteUser(id)"
            ></gestionUsuario>
        </div>
    </div>
    
</template>

<script>
import { SHA256 } from 'crypto-js';

import login from './Login.vue';
import sidebar from './Sidebar.vue';

import ticketList from './Ticket/List.vue';
import ticketView from './Ticket/View.vue';
import ticketForm from './Ticket/Form.vue';

import status from './Status.vue';
import priority from './Priority.vue';

import gestionDatos from './GestionDatos.vue';

import listaUsuariosAdmin from './Admin/ListaUsuarios.vue';
import gestionUsuario from './GestionUsuario.vue';

export default {
    components: {
        login,
        sidebar,
        ticketList,
        ticketForm,
        ticketView,
        status,
        priority,
        gestionDatos,
        listaUsuariosAdmin,
        gestionUsuario
    },
    data() {
        return {
            activePage: 0,
            isLogged: false,
            loggedUser: { id: -1, username: 'null', password: 'null', role: -1, isFirst: 0 },
            ticketList: [],
            listaEstatus: [],
            listaPrioridades: [],
            userList: [],
            assignableUsers: [],
            selectedTicket: {},
            isTicket: true,
            usersList: [],
            allUsers: [],
            createNewUser: false
        }
    },
    methods: {
        changeFront(){
            this.isTicket = !this.isTicket;
            if (this.isTicket) {
                this.activePage = 1
            } else {
                this.activePage = 5
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
            this.loggedUser = {
                id: resData.ID_usuario,
                username: resData.nomusua,
                password: resData.Contraseña,
                role: resData.rol,
                isFirst: resData.justCreated.data[0]
            } 
        },
        findAssignableUsers() {
            this.assignableUsers = [];
            this.userList.forEach((user) => {
                if (Number(user.rol) > 0) {
                    this.assignableUsers.push(user);
                }
            });
        },
        async tryLogIn(user, pass) {
            var encryptedPass = SHA256(pass + process.env.VUE_APP_SECRET).toString();
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/logindata/?username=${user}&password=${encryptedPass}`).then(
                async (response) => {
                    if (JSON.stringify(response.data[0]) == JSON.stringify([])) {
                        alert("Usuario o contraseña incorrectos");
                    } else {
                        this.dataToLoggedUser(response.data[0][0]);
                        this.isLogged = true;                        
                        if (this.loggedUser.isFirst == 1) {
                            this.isTicket = false;
                            this.loadData(5);
                        } else {
                            this.loadData(1);
                        }
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
                case 5:
                    await this.getUsers();
                    this.usersToDisplay();
                    this.activePage = page;
                    break;
                case 6:
                    await this.getUsers();
                    this.usersToDisplay();
                    this.activePage = page;
                    break;
            }
        },
        usersToDisplay() {
            this.usersList = [];
            let usr = this.allUsers.find((obj) => obj.ID_usuario == this.loggedUser.id);
            this.allUsers.splice(this.allUsers.indexOf(usr),1);
            this.usersList.push(...this.allUsers);
        },
        async getTickets() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/tickets`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.ticketList = response.data[0];
                    }
                }
            )
        },
        async getusernames() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/tickets/usernames`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.userList = response.data[0];
                        this.findAssignableUsers();
                    }
                }
            )
        },
        async getPriority() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaPrioridades = response.data[0];
                    }
                }
            )
        },
        async getStatus() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.listaEstatus = response.data[0];
                    }
                }
            )
        },
        async getUsers() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_USER}/admin/allusers/`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.allUsers = response.data[0];
                    }
                }
            )
        },
        async createUser(id, user, isCreate, page){
            if (user.username == '' || user.role == -1 || (isCreate && user.newPassword == '')) {
                alert("Ingrese todos los campos");
            } else if (isCreate){
                await axios.post(`${process.env.VUE_APP_BACKENDURL_USER}/admin/allusers/`, user).then(
                    async (response) => {
                        await this.loadData(page);
                        return alert(`Usuario creado con exito`);
                    }).catch((error) =>{
                        return alert(`Ya existe usuario con username: ${user.username}`);
                    })
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL_USER}/admin/allusers/?id=${id}`, user).then(
                    async (response) => {
                        await this.loadData(page);
                        return alert(`Usuario actualizado con exito`);
                    }).catch((error) =>{
                        return alert(error);
                    })
            }
        },
        async deleteUser(id){
            await axios.put(`${process.env.VUE_APP_BACKENDURL_USER}/admin/allusers/delete/?id=${id}`).then(
                (response) => {
                    this.loadData(6);
                    return alert(`Usuario eliminado con exito`);
                }).catch((error) => {
                    return alert(error);
                })
        },
        getSelectedListIndex(){
            let index = this.$refs.adminUserList.activeListItem;
            if (index == -1) {
                return '';
            } else {
                let fixedList = JSON.parse(JSON.stringify(this.usersList))
                return fixedList[index];
            }
        }
    }
}
</script>