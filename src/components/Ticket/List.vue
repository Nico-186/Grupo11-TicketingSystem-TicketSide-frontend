<template>
    <div class="d-flex h-100 flex-column p-0 m-0 gap-4">
        <div>
            <label>Buscar ticket</label>
            <input type="text" class="form-control" placeholder="Búsqueda">
        </div>
        <div class="flex-grow-1 d-flex flex-column border w-100" style="background-color: #171a1c; min-height: 0;">
            <p class="w-100 mb-0 px-3 py-2 fs-5">Tickets</p>
            <div class="list-group">
                <div class="d-flex" :class="ticketList.length >= 11 ? 'pe-3' : 'pe-0'">
                    <li class="list-group-item border" style="width: 3.5%;">Id</li>
                    <li v-for="header in headers" class="list-group-item border" style="width: 16.1%;">{{ header }}</li>
                </div>
            </div>
            <div class="flex-grow-1 w-100 overflow-auto">
                <div class="list-group">
                    <ul class="list-group">
                        <button v-if="ticketList.length != 0" v-for="ticket in ticketList" :id="ticket.ID_ticket" type="button"
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
                        <div v-if="ticketList.length == 0" class="d-flex w-100">
                            <li class="list-group-item border w-100">No existen tickets</li>
                         </div>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ["ticketList", "statusList", "priorityList", "userList", "openSelectedTicket"],
    data() {
        return {
            headers: ['Asunto', 'Estatus', 'Prioridad', 'Fecha de publicación', 'Creado por', 'Encargado']
        }
    },
    methods: {
        findStatusById(id) {
            return this.statusList.find((obj) => obj.ID_status == id)
        },
        findPriorityById(id) {
            return this.priorityList.find((obj) => obj.ID_prio == id)
        },
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id)
        }
    }
}
</script>