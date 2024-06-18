<template>
    <div class="container-fluid w-100 h-100 p-4">
        <div class="d-flex h-100 flex-column p-0 m-0 gap-3">
            <div class="form-group">
                <label>Buscar ticket</label>
                <input type="text" class="form-control" placeholder="Busqueda">
            </div>



            <div class="flex-grow-1 d-flex flex-column border w-100" style="background-color: #171a1c; min-height: 0;">

                <p class="w-100 mb-0 px-3 py-2 fs-5">Tickets</p>
                
                <div class="list-group">
                    <div class="d-flex" :class="tickets.length >= 11 ? 'pe-3' : 'pe-0'">
                        <li class="list-group-item border" style="width: 3.5%;">Id</li>
                        <li v-for="header in headers" class="list-group-item border" style="width: 16.1%;">{{ header }}</li>
                    </div>
                </div>

                <div class="flex-grow-1 w-100 overflow-auto">
                    <div class="list-group">
                        <ul class="list-group">
                            <button 
                                v-for="ticket in tickets"
                                :id="ticket.ID_ticket" 
                                type="button"
                                class="list-group-item list-group-item-action p-0"
                                :class="{ active: activeListItem == ticket.ID_ticket}"
                                @click.prevent="openSelectedTicket(ticket.ID_ticket)">
                                <div class="d-flex ">
                                    <li class="list-group-item border" style="width: 3.5%;">{{ticket.ID_ticket}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{ticket.NAME}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{findStatusById(ticket.IDstatus)}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{findPriorityById(ticket.IDprio)}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{new Date(ticket.Fecha).toLocaleDateString()}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{findUserById(ticket.IDusuario)}}</li>
                                    <li class="list-group-item border" style="width: 16.1%;">{{ticket.IDencargado}}</li>
                                </div>
                            </button>
                        </ul>
                    </div>
                </div>
            </div>
        </div>


    </div>
</template>

<script>
export default {
    props: ["tickets","status","prioridades","userList","openSelectedTicket"],
    data() {
        return {
            activeListItem: -1,
            idSelectedItem: -1,
            headers: ['Nombre','Estatus','Prioridad','Fecha','Creado por','Encargado']
        }
    },
    methods: {
        changeActiveListItem(index) {
            console.log(this.prioridades)
            console.log(this.status)
            if (this.activeListItem == index){
                this.activeListItem = -1;
            } else {
                this.activeListItem = index;
            };
        },
        findStatusById(id) {
            return this.status.find((obj) => obj.ID_status == id).tipostatus
        },
        findPriorityById(id) {
            return this.prioridades.find((obj) => obj.ID_prio == id).tipoprio
        },
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id).nomusua
        }
    }
}
</script>