<template>
    <div class="container-fluid h-100 v-100 py-4" style="padding-left:15%; padding-right: 15%">
        <div class="d-flex flex-column align-items-center">
            <div class="d-flex justify-content-center m-2 w-100">
                <h4 class="w-auto">Crear nuevo ticket</h4>
            </div>
            <div class="form-group mb-4 w-100">
                <label>Asunto</label>
                <input v-model="ticketName" type="text" class="form-control" placeholder="Asunto del ticket">
            </div>
            <div class="form-group mb-4 w-100">
                <label>Prioridad del ticket</label>
                <select v-model="ticketPriority" class="text-reset btn btn-outline-secondary text-start w-100" style="background-color: #212529" placeholder="MIAU">
                    <option value="" disabled selected>Select your option</option>
                    <option v-for="prioridad in prioridades" style="background-color: #212529;" :value="prioridad.ID_prio">{{prioridad.tipoprio}}</option>
                </select>
            </div>
            <div class="form-group mb-4 w-100">
                <label>Descripción</label>
                <textarea v-model="ticketDescription" type="text" class="form-control"
                    placeholder="Descripción del problema" rows="8"></textarea>
            </div>
            <button type="button" class="btn btn-success mb-3 w-100" @click.prevent="CreateTicket()">Crear
                ticket</button>
        </div>
    </div>
</template>

<script>
export default {
    props: ["prioridades","idLog"],
    data() {
        return {
            ticketName: "",
            ticketDescription: "",
            ticketPriority: ""
        }
    },
    methods: {
        async CreateTicket() {
            await axios.post(`http://localhost:3000/tickets/`, { id: this.idLog, name: this.ticketName, description: this.ticketDescription, priority: this.ticketPriority, date: moment().format().slice(0, 19).replace('T', ' ') }).then(
                    (response) => {
                        return alert(`Usuario creado con exito`);
                    }).catch((error) =>{
                        return alert(`Ya existe usuario con username`);
                    })
        },
        changeActiveListItem(index) {
            this.ticketPriority = index;
        }
    }
}
</script>