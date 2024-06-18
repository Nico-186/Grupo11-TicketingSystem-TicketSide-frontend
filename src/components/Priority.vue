<template>
    <div class="d-flex flex-column justify-content-center align-items-center gap-3 w-100 h-100 py-4"
        style="padding-left:5%; padding-right: 5%">
        <div class="d-flex flex-column align-items-center gap-3 w-50 h-auto">
            <div class="h4 d-flex pe-2 m-0">Gestionar Prioridades</div>

            <div class="d-flex flex-column w-100">
                <label>Editar prioridad</label>
                <select v-model="selectedPriorityID" class="text-reset btn btn-outline-secondary text-start"
                    style="background-color: #212529">
                    <option :value="-1" selected @click="setPriorityName('')">Crear prioridad</option>
                    <option v-for="prioridad in priorityList" style="background-color: #212529;"
                        :value="prioridad.ID_prio" @click="setPriorityName(prioridad.tipoprio)">
                        #{{ prioridad.ID_prio }}-{{ prioridad.tipoprio }}
                    </option>
                </select>
            </div>

            <div class="form-group w-100">
                <label>Nombre de la prioridad</label>
                <input v-model="selectedPriorityName" type="text" class="form-control" placeholder="Asunto del ticket">
            </div>

            <button type="button" class="btn btn-success w-100" @click="postPriority()">{{ selectedPriorityID == -1 ?
                'Crear Prioridad' :
                'Guardar prioridad' }}</button>
            <button type="button" class="btn btn-danger w-50"
                :style="{ visibility: selectedPriorityID == -1 ? 'hidden' : 'visible' }"
                @click="deletePriority()">Eliminar Prioridad</button>
        </div>
    </div>
</template>

<script>
export default {
    props: ["prioridades"],
    data() {
        return {
            selectedPriorityID: -1,
            selectedPriorityName: '',
            priorityList: []
        }
    },
    mounted() {
        this.priorityList = this.prioridades;
    },
    methods: {
        setPriorityName(name) {
            this.selectedPriorityName = name;
        },
        getPriority() {
            axios.get(`http://localhost:3000/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.priorityList = response.data;
                    }
                }
            )
        },
        async postPriority() {
            if (this.selectedPriorityID == -1) {
                await axios.post(`http://localhost:3000/priority/`, { name: this.selectedPriorityName }).then(
                    (response) => {
                        this.selectedPriorityID= -1;
                        this.selectedPriorityName = '';
                        this.getPriority();
                        return alert(`Ticket actualizado con exito`);
                    }).catch((error) =>{
                        return alert(error);
                    })
            } else {
                await axios.put(`http://localhost:3000/priority/?id=${this.selectedPriorityID}`, { name: this.selectedPriorityName }).then(
                    (response) => {
                        this.selectedPriorityID= -1;
                        this.selectedPriorityName = '';
                        this.getPriority();
                        return alert(`Ticket actualizado con exito`);
                    }).catch((error) =>{
                        return alert(error);
                    })
            }  
        },
        async deletePriority() {
            await axios.delete(`http://localhost:3000/priority/?id=${this.selectedPriorityID}`).then(
                (response) => {
                    this.selectedPriorityID = -1;
                    this.selectedPriorityName = '';
                    this.getPriority();
                    return alert(`Ticket boorrao con exito`);
                }).catch((error) => {
                    return alert(error);
                })
        }
    }
}
</script>