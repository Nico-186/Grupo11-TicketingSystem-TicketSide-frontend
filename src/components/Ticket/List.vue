<template>
    <div class="d-flex h-100 flex-column p-0 m-0 gap-4">
        <div>
            <label>Buscar ticket</label>
            <input v-model="searchText" type="text" class="form-control" placeholder="Búsqueda"/>
            <div class="d-flex gap-2 pt-2">
                <select v-model="selectedStatusID"
                class="text-reset btn btn-outline-secondary text-start w-25"
                style="background-color: #212529">
                    <option :value="-1" selected>Estatus</option>
                    <option v-for="status in statusList"
                    style="background-color: #212529;"
                    :value="status.ID_status">
                        {{ status.tipostatus }}
                    </option>
                </select>
                <select v-model="selectedPriorityID"
                class="text-reset btn btn-outline-secondary text-start w-25"
                style="background-color: #212529">
                    <option :value="-1" selected>Prioridad</option>
                    <option v-for="prioridad in priorityList"
                    style="background-color: #212529;"
                    :value="prioridad.ID_prio">
                        {{ prioridad.tipoprio }}
                    </option>
                </select>
                <select v-model="userID"
                class="text-reset btn btn-outline-secondary text-start w-25"
                style="background-color: #212529">
                    <option :value="-1" selected>Creado por</option>
                    <option v-for="user in userList"
                    style="background-color: #212529;"
                    :value="user.ID_usuario">
                        {{ user.nomusua }}
                    </option>
                </select>
                <select v-model="managerID"
                class="text-reset btn btn-outline-secondary text-start w-25"
                style="background-color: #212529">
                    <option :value="-1" selected>Encargado</option>
                    <option v-for="encargado in assignableUsers"
                    style="background-color: #212529;"
                    :value="encargado.ID_usuario">
                        {{ encargado.nomusua }}
                    </option>
                </select>
                <button class="btn btn-secondary" @click="doSearch()">Buscar</button>
            </div>
        </div>
        <div class="flex-grow-1 d-flex flex-column border w-100" style="background-color: #171a1c; min-height: 0;">
            <p class="w-100 mb-0 px-3 py-2 fs-5">Tickets</p>
            <div class="list-group">
                <div class="d-flex" :class="ticketsToShow.length >= 11 ? 'pe-3' : 'pe-0'">
                    <li class="list-group-item border" style="width: 3.5%;">Id</li>
                    <li v-for="header in headers" class="list-group-item border" style="width: 16.1%;">{{ header }}</li>
                </div>
            </div>
            <div class="flex-grow-1 w-100 overflow-auto">
                <div class="list-group">
                    <ul class="list-group">
                        <button v-if="ticketsToShow.length != 0" v-for="ticket in ticketsToShow" :id="ticket.ID_ticket" type="button"
                            class="list-group-item list-group-item-action p-0"
                            @click.prevent="openSelectedTicket(ticket.ID_ticket)">
                            <div class="d-flex bg-transparent">
                                <li class="list-group-item border bg-transparent" style="width: 3.5%;">{{ ticket.ID_ticket }}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;">{{ ticket.NAME }}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;">{{ (findStatusById(ticket.IDstatus)).tipostatus }}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;">{{ (findPriorityById(ticket.IDprio)).tipoprio }}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;">{{ new Date(ticket.Fecha).toLocaleDateString()}}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;">{{ (findUserById(ticket.IDusuario)).nomusua }}</li>
                                <li class="list-group-item border bg-transparent" style="width: 16.1%;" :style="{ opacity: ticket.IDencargado == null ? '0.5' : '1'}">
                                    {{ ticket.IDencargado == null ? 'Sin asignar' : (findUserById(ticket.IDencargado)).nomusua }}
                                </li>
                            </div>
                        </button>
                        <div v-if="ticketsToShow.length == 0" class="d-flex w-100">
                            <li class="list-group-item border w-100">No existen tickets para mostrar</li>
                         </div>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ["ticketList", "statusList", "priorityList", "userList", "assignableUsers", "openSelectedTicket", "loggedUser"],
    data() {
        return {
            headers: ['Asunto', 'Estatus', 'Prioridad', 'Fecha de publicación', 'Creado por', 'Encargado'],
            ticketsToShow: [],
            searchText: '',
            selectedPriorityID: -1,
            selectedStatusID: -1,
            userID: -1,
            managerID: -1
        }
    },
    mounted() {
        this.ticketsToShow = this.roleBasedList(this.ticketList);
    },
    methods: {
        async doSearch() {
            let urlParams = [];
            
            if(this.searchText.trim() == '' && this.selectedPriorityID == -1 && this.selectedStatusID == -1 && this.userID == -1 && this.managerID == -1){
                this.$parent.activePage = 0;
                this.$parent.loadData(1);
            }

            if (this.searchText.trim() != '') {
                urlParams.push(`text=${this.searchText}`);
            }
            if (this.selectedPriorityID != -1) {
                urlParams.push(`idPrioridad=${this.selectedPriorityID}`);
            }
            if (this.selectedStatusID != -1) {
                urlParams.push(`idEstatus=${this.selectedStatusID}`);
            }
            if (this.userID != -1) {
                urlParams.push(`idUsuario=${this.userID}`);
            }
            if (this.managerID != -1) {
                urlParams.push(`idEncargado=${this.managerID}`);
            }
            await axios.get('https://busqueda.azurewebsites.net/api/busqueda/?'.concat(urlParams.join('&'))).then(
                (response) => {
                    this.searchText = '';
                    this.selectedPriorityID = -1;
                    this.selectedStatusID = -1;
                    this.userID = -1;
                    this.managerID = -1; 
                    if(response.data.length != 0) {
                        if (this.roleBasedList(response.data) == 0) {
                            return alert('No se han encontrado tickets con los parámetros de búsqueda');
                        } else {
                            this.ticketsToShow = this.roleBasedList(response.data);
                        }
                    } else {
                        return alert('No se han encontrado tickets con los parámetros de búsqueda');
                    }
                }
            )
        },
        findStatusById(id) {
            return this.statusList.find((obj) => obj.ID_status == id)
        },
        findPriorityById(id) {
            return this.priorityList.find((obj) => obj.ID_prio == id)
        },
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id)
        },
        roleBasedList(list) {
            if (this.loggedUser.role == 0){
                return list.filter(this.isFromUserLogged);
            } else {
                return list;
            }
        },
        isFromUserLogged(ticket){
            return ticket.IDusuario == this.loggedUser.id;
        }
    }
}
</script>