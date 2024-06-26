<template>
        <div class="d-flex flex-column align-items-center gap-4 mt-2" style="padding-left:15%; padding-right: 15%">

            <div class="d-flex justify-content-center w-100">
                <h4 class="w-auto">Publicar nuevo ticket</h4>
            </div>

            <div class="w-100">
                <label>Asunto del ticket</label>
                <input v-model="ticketName" type="text" class="form-control" placeholder="Asunto del ticket">
            </div>

            <div class="w-100">
                <label>Prioridad del ticket</label>
                <select 
                v-model="ticketPriority" 
                class="text-reset btn btn-outline-secondary text-start w-100" 
                style="background-color: #212529">
                    <option value="" disabled selected>Seleccione una opción</option>
                    <option v-for="prioridad in priorityList" style="background-color: #212529;" :value="prioridad.ID_prio">{{prioridad.tipoprio}}</option>
                </select>
            </div>

            <div class="w-100">
                <label>Descripción</label>
                <textarea v-model="ticketDescription" type="text" class="form-control"
                    placeholder="Descripción del problema" rows="8">
                </textarea>
            </div>
            <button type="button"
            class="btn btn-success w-100"
            :disabled="ticketName.trim() == '' || ticketDescription.trim() == '' || ticketPriority == ''"
            @click.prevent="CreateTicket()">
                Crear ticket
            </button>
        </div>
</template>

<script>
export default {
    props: ["priorityList","idLog"],
    data() {
        return {
            ticketName: "",
            ticketDescription: "",
            ticketPriority: ""
        }
    },
    methods: {
        async CreateTicket() {
            await axios.post(`${process.env.VUE_APP_BACKENDURL_TICKET}/tickets/`, { id: this.idLog, name: this.ticketName.trim(), description: this.ticketDescription.trim(), priority: this.ticketPriority, date: new Date().toISOString().replace('T', ' ').replace('Z', ' ') }).then(
                    async (response) => {
                        await this.$parent.getTickets();
                        await axios.post(`${process.env.VUE_APP_BACKENDURL_TICKET}/time/`, { status: 1, start: new Date().toISOString().replace('T', ' ').replace('Z', ' '), ticket: (this.$parent.ticketList[this.$parent.ticketList.length - 1]).ID_ticket }).then(
                            (response) => {
                                alert(`Ticket publicado con éxito`);
                                this.ticketName = "";
                                this.ticketDescription = "";
                                this.ticketPriority = "";
                            }).catch((error) =>{
                                return alert(error);
                            })
                    }).catch((error) =>{
                        return alert(error);
                    })
        }
    }
}
</script>