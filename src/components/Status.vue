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
                <option v-for="stat in statusList"
                style="background-color: #212529;"
                :value="stat.ID_status"
                @click="setStatusName(stat.tipostatus)">
                    #{{ stat.ID_status }}-{{ stat.tipostatus }}
                </option>
            </select>
        </div>

        <div class="form-group w-50">
            <label>Nombre del estatus</label>
            <input v-model="selectedStatusName" type="text" class="form-control" placeholder="Nombre del estatus">
        </div>
        
        <div class="d-flex flex-column align-items-center w-50 gap-2">
            <button type="button"
            class="btn btn-success w-100" 
            @click.prevent="postStatus()">
                {{ selectedStatusID == -1 ? 'Crear estatus' : 'Guardar cambios' }}
            </button>
            <button type="button"
            class="btn btn-danger w-100"
            :style="{ visibility: selectedStatusID == -1 ? 'hidden' : 'visible' }"
            @click="deleteComment()">
                Eliminar estatus
            </button>    
        </div>
    </div>
</template>

<script>
export default {
    props: ["allStatus"],
    data() {
        return {
            selectedStatusID: -1,
            selectedStatusName: '',
            statusList: []
        }
    },
    mounted() {
        this.statusList = this.allStatus;
    },
    methods: {
        setStatusName(name) {
            this.selectedStatusName = name;
        },
        getStatus() {
            axios.get(`${process.env.VUE_APP_BACKENDURL}/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.statusList = response.data;
                    }
                }
            )
        },
        async postStatus() {
            if (this.selectedStatusID == -1) {
                await axios.post(`${process.env.VUE_APP_BACKENDURL}/status/`, { name: this.selectedStatusName }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.getStatus();
                        return alert(`Estatus creado con éxito`);
                    }).catch((error) =>{
                        return alert('Ha surgido un error al crear el estatus');
                    })
            } else {
                await axios.put(`${process.env.VUE_APP_BACKENDURL}/status/?id=${this.selectedStatusID}`, { name: this.selectedStatusName }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.getStatus();
                        return alert(`Estatus actualizado con éxito`);
                    }).catch((error) =>{
                        return alert('Ha surgido un error al actualizar el estatus');
                    })
            }  
        },
        async deleteStatus() {
            await axios.delete(`${process.env.VUE_APP_BACKENDURL}/status/?id=${this.selectedStatusID}`).then(
                (response) => {
                    this.selectedStatusID = -1;
                    this.selectedStatusName = '';
                    this.getStatus();
                    return alert(`Estatus eliminado con éxito`);
                }).catch((error) => {
                    return alert('Ha surgido un error al actualizar el estatus');
                })
        }
    }
}
</script>