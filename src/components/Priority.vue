<template>
    <div class="d-flex flex-column align-items-center gap-4 mt-2">

        <div class="d-flex justify-content-center w-50">
            <h4 class="w-auto">Gestionar Prioridades</h4>
        </div>

        <div class="d-flex flex-column w-50">
            <label>{{ selectedPriorityName == '' ? 'Crear nueva prioridad' : 'Editar prioridad seleccionada' }}</label>
            <select v-model="selectedPriorityID"
            class="text-reset btn btn-outline-secondary text-start"
            style="background-color: #212529">
                <option :value="-1" selected @click="setPriorityName('')">Crear prioridad</option>
                <option v-for="prioridad in priorityList"
                style="background-color: #212529;"
                :value="prioridad.ID_prio"
                @click="setPriorityName(prioridad.tipoprio)">
                    #{{ prioridad.ID_prio }}-{{ prioridad.tipoprio }}
                </option>
            </select>
        </div>

        <div class="form-group w-50">
            <label>Nombre de la prioridad</label>
            <input v-model="selectedPriorityName" type="text" class="form-control" placeholder="Nombre de la prioridad">
        </div>

        <div class="d-flex flex-column align-items-center w-50 gap-2">
            <button type="button"
            class="btn btn-success w-100"
            :disabled="selectedPriorityName.trim() == '' || selectedPriorityName.trim() == oldPriorityName"
            @click="postPriority()">
                {{ selectedPriorityID == -1 ? 'Crear Prioridad' : 'Guardar prioridad' }}
            </button>
            <button type="button"
            class="btn btn-danger w-100"
            :style="{ visibility: selectedPriorityID == -1 ? 'hidden' : 'visible' }"
            @click="deletePriority()">
                Eliminar Prioridad
            </button>
        </div>
    </div>
</template>

<script>
export default {
    props: ["allPriorities","allTickets"],
    data() {
        return {
            oldPriorityName: '',
            selectedPriorityID: -1,
            selectedPriorityName: '',
            priorityList: []
        }
    },
    mounted() {
        this.priorityList = this.allPriorities;
    },
    methods: {
        setPriorityName(name) {
            this.selectedPriorityName = name;
            this.oldPriorityName = name;
        },
        getPriority() {
            axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/priority`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.priorityList = response.data[0];
                    }
                }
            )
        },
        async postPriority() {
            if (this.selectedPriorityID == -1) {
                await axios.post(`${process.env.VUE_APP_BACKENDURL_TICKET}/priority/`, { name: this.selectedPriorityName }).then(
                    (response) => {
                        this.selectedPriorityID = -1;
                        this.selectedPriorityName = '';
                        this.getPriority();
                        return alert(`Prioridad creada con éxito`);
                    }).catch((error) => {
                        return alert('Ha surgido un error al crear la prioridad');
                    })
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/priority/?id=${this.selectedPriorityID}`, { name: this.selectedPriorityName }).then(
                    (response) => {
                        this.selectedPriorityID = -1;
                        this.selectedPriorityName = '';
                        this.getPriority();
                        return alert(`Prioridad actualizada con éxito`);
                    }).catch((error) => {
                        return alert('Ha surgido un error al actualizad la prioridad');
                    })
            }
        },
        async deletePriority() {
            let ticketUsingPriority = this.allTickets.find((obj) => obj.IDprio == this.selectedPriorityID)
            if(ticketUsingPriority != undefined){
                return alert('No se puede eliminar, puesto que al menos un ticket es de esta prioridad');
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/priority/delete/?id=${this.selectedPriorityID}`).then(
                    (response) => {
                        this.selectedPriorityID = -1;
                        this.selectedPriorityName = '';
                        this.getPriority();
                        return alert(`Prioridad eliminada con éxito`);
                    }).catch((error) => {
                        return alert('Ha surgido un error al eliminar la prioridad');
                    })
            }
        }
    }
}
</script>