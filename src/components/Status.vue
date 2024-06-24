<template>
    <div class="d-flex flex-column align-items-center gap-4 mt-2">

        <div class="d-flex justify-content-center w-50">
            <h4 class="w-auto">Gestionar Estatus</h4>
        </div>

        <div class="d-flex flex-column w-50">
            <label>{{ selectedStatusName == '' ? 'Crear nuevo estatus' : 'Editar estatus seleccionado' }}</label>
            <select v-model="selectedStatusID"
            class="text-reset btn btn-outline-secondary text-start"
            style="background-color: #212529">
                <option :value="-1"selected @click="setStatusName('')">Crear nuevo estatus</option>
                <option v-for="status in statusList"
                style="background-color: #212529;"
                :value="status.ID_status"
                @click="setStatusName(status.tipostatus), setIsFinished(status.statusFinal.data[0] == 1)">
                    #{{ status.ID_status }}-{{ status.tipostatus }}
                </option>
            </select>
        </div>

        <div class="d-flex flex-column w-50">
            <label>Nombre del estatus</label>
            <div class="d-flex gap-3">
                <div class="flex-grow-1">
                    <input v-model="selectedStatusName" type="text" class="form-control" placeholder="Nombre del estatus">
                </div>
                <div v-if="selectedStatusID != 1" class="d-flex gap-2 align-items-center">
                    <label class="form-check-label" for="flexCheckDefault">
                        Ticket finalizado?
                    </label>
                    <input v-model="isFinished" class="form-check-input" type="checkbox">
                </div>
            </div>
        </div>
        
        <div class="d-flex flex-column align-items-center w-50 gap-2">
            <button type="button"
            class="btn btn-success w-100"
            :disabled="!((selectedStatusName.trim() != '') && (selectedStatusName.trim() != oldStatusName || isFinished != oldIsFinished))"
            @click.prevent="postStatus()">
                {{ selectedStatusID == -1 ? 'Crear estatus' : 'Guardar cambios' }}
            </button>
            <button type="button"
            class="btn btn-danger w-100"
            :style="{ visibility: selectedStatusID <= 1 ? 'hidden' : 'visible' }"
            @click="deleteStatus()">
                Eliminar estatus
            </button>    
        </div>
    </div>
</template>

<script>
export default {
    props: ["allStatus","allTickets"],
    data() {
        return {
            oldStatusName: '',
            oldIsFinished: '',
            selectedStatusID: -1,
            selectedStatusName: '',
            statusList: [],
            isFinished: false
        }
    },
    mounted() {
        this.statusList = this.allStatus;
    },
    methods: {
        setStatusName(name) {
            this.selectedStatusName = name;
            this.oldStatusName = name;
        },
        getStatus() {
            axios.get(`${process.env.VUE_APP_BACKENDURL}/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.statusList = response.data[0];
                    }
                }
            )
        },
        setIsFinished(bool) {
            this.isFinished = bool;
            this.oldIsFinished = bool;
        },
        async postStatus() {
            if (this.selectedStatusID == -1) {
                await axios.post(`${process.env.VUE_APP_BACKENDURL}/status/`, { name: this.selectedStatusName.trim(), isfinal: this.isFinished ? 1 : 0 }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.isFinished = false;
                        this.getStatus();
                        return alert(`Estatus creado con éxito`);
                    }).catch((error) =>{
                        return alert('Ha surgido un error al crear el estatus');
                    })
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL}/status/?id=${this.selectedStatusID}`, { name: this.selectedStatusName.trim(), isfinal: this.isFinished ? 1 : 0 }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.isFinished = false;
                        this.getStatus();
                        return alert(`Estatus actualizado con éxito`);
                    }).catch((error) =>{
                        return alert('Ha surgido un error al actualizar el estatus');
                    })
            }  
        },
        async deleteStatus() {
            let ticketUsingStatus = this.allTickets.find((obj) => obj.IDstatus == this.selectedStatusID)
            if(ticketUsingStatus != undefined){
                return alert('No se puede eliminar, puesto que al menos un ticket se encuentra actualmente en este estado');
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL}/status/delete/?id=${this.selectedStatusID}`).then(
                    (response) => {
                        this.selectedStatusID = -1;
                        this.selectedStatusName = '';
                        this.isFinished = false;
                        this.getStatus();
                        return alert(`Estatus eliminado con éxito`);
                    }).catch((error) => {
                        return alert('Ha surgido un error al actualizar el estatus');
                    })
            }
        }
    }
}
</script>