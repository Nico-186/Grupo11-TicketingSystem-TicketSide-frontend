<template>
    <div class="container-fluid h-100 v-100 py-4" style="padding-left:5%; padding-right: 15%">
        <div class="d-flex flex-column align-items-center gap-3">

            <div class="d-flex align-items-center w-100">
                <div class="h4 d-flex pe-2 m-0"># {{ ticket.ID_ticket }}</div>
                <div class="h4 d-flex flex-grow-1 m-0">- {{ ticket.NAME }}</div>

                <div class="d-flex form-group flex-column w-auto">
                    <label>Encargado:</label>

                    <p v-if="viewRoleBased() == 0" class="form-control m-0">AAAAA</p>

                    <button v-if="viewRoleBased() == 1" class="btn btn-secondary">AAAAA</button>

                    <select v-if="viewRoleBased() == 2" v-model="ticketPriority"
                        class="text-reset btn btn-outline-secondary text-start" style="background-color: #212529"
                        placeholder="MIAU">
                        <option value="" disabled selected>Select your option</option>
                        <option v-for="prioridad in prioridades" style="background-color: #212529;"
                            :value="prioridad.ID_prio">
                            {{prioridad.tipoprio}}
                        </option>
                    </select>

                </div>
            </div>

            <div class="d-flex justify-content-center w-100 gap-3">
                <div class="form-group w-50">
                    <label>Creado por:</label>
                    <p class="form-control">{{ findUserById(ticket.IDusuario) }}</p>
                </div>
                <div class="form-group w-50">
                    <label>Fecha de creación:</label>
                    <p class="form-control">{{ new Date(ticket.Fecha).toLocaleDateString() }}</p>
                </div>
            </div>

            <div class="d-flex justify-content-center w-100 gap-3">
                <div class="w-50">
                    <label>Prioridad del ticket</label>
                    <select v-model="ticketPriority" class="text-reset btn btn-outline-secondary text-start w-100"
                        style="background-color: #212529" placeholder="MIAU">
                        <option value="" disabled selected>Select your option</option>
                        <option v-for="prioridad in prioridades" style="background-color: #212529;"
                            :value="prioridad.ID_prio">
                            {{ prioridad.tipoprio }}</option>
                    </select>
                </div>
                <div class="w-50">
                    <label>Estatus del ticket</label>
                    <select v-model="ticketPriority" class="text-reset btn btn-outline-secondary text-start w-100"
                        style="background-color: #212529" placeholder="MIAU">
                        <option value="" disabled selected>Select your option</option>
                        <option v-for="prioridad in prioridades" style="background-color: #212529;"
                            :value="prioridad.ID_prio">
                            {{ prioridad.tipoprio }}</option>
                    </select>
                </div>
            </div>

            <div class="form-group w-100">
                <label>Descripción</label>
                <textarea type="text" class="form-control" placeholder="Descripción del problema" rows="8" disabled
                    :value="ticket.Descripcion"></textarea>
            </div>

        </div>
    </div>
</template>

<script>
export default {
    props: ["ticket","loggedUser","status","prioridades","userList"],
    data() {
        return {
            isMiau: false
        }
    },
    methods: {
        viewRoleBased() {
            switch (this.loggedUser.role){
                case '0':
                    return 0;
                case '1':
                    if (this.ticket.IDencargado != this.loggedUser.id && this.ticket.IDencargado != null) {
                        return 0;
                    }
                    return 1;
                case '2':
                    return 2;
            }
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