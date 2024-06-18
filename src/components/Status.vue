<template>
    <div class="d-flex flex-column justify-content-center align-items-center gap-3 w-100 h-100 py-4"
        style="padding-left:5%; padding-right: 5%">
        <div class="d-flex flex-column align-items-center gap-3 w-50 h-auto">
            <div class="h4 d-flex pe-2 m-0">Gestionar Status</div>

            <div class="d-flex flex-column w-100">
                <label>Editar status</label>
                <select v-model="selectedStatusID" class="text-reset btn btn-outline-secondary text-start"
                    style="background-color: #212529">
                    <option :value="-1" selected @click="setStatusName('')">Crear status</option>
                    <option v-for="stat in statusList" style="background-color: #212529;" :value="stat.ID_status"
                        @click="setStatusName(stat.tipostatus)">
                        #{{ stat.ID_status }}-{{ stat.tipostatus }}
                    </option>
                </select>
            </div>

            <div class="form-group w-100">
                <label>Nombre del estatus</label>
                <input v-model="selectedStatusName" type="text" class="form-control" placeholder="Asunto del ticket">
            </div>

            <button type="button" class="btn btn-success w-100" @click.prevent="postStatus()">{{ selectedStatusID == -1
                ? 'Crear estatus' :
                'Guardar estatus' }}</button>
            <button type="button" class="btn btn-danger w-50"
                :style="{ visibility: selectedStatusID == -1 ? 'hidden' : 'visible' }" @click="deleteComment()">Eliminar
                estatus</button>
        </div>
    </div>
</template>

<script>
export default {
    props: ["status"],
    data() {
        return {
            selectedStatusID: -1,
            selectedStatusName: '',
            statusList: []
        }
    },
    mounted() {
        this.statusList = this.status;
    },
    methods: {
        setStatusName(name) {
            this.selectedStatusName = name;
        },
        getStatus() {
            axios.get(`http://localhost:3000/status`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.statusList = response.data;
                    }
                }
            )
        },
        async postStatus() {
            if (this.selectedStatusID == -1) {
                await axios.post(`http://localhost:3000/status/`, { name: this.selectedStatusName }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.getStatus();
                        return alert(`Ticket actualizado con exito`);
                    }).catch((error) =>{
                        return alert(error);
                    })
            } else {
                await axios.put(`http://localhost:3000/status/?id=${this.selectedStatusID}`, { name: this.selectedStatusName }).then(
                    (response) => {
                        this.selectedStatusID= -1;
                        this.selectedStatusName = '';
                        this.getStatus();
                        return alert(`Ticket actualizado con exito`);
                    }).catch((error) =>{
                        return alert(error);
                    })
            }  
        },
        async deleteStatus() {
            await axios.delete(`http://localhost:3000/status/?id=${this.selectedStatusID}`).then(
                (response) => {
                    this.selectedStatusID = -1;
                    this.selectedStatusName = '';
                    this.getStatus();
                    return alert(`Ticket boorrao con exito`);
                }).catch((error) => {
                    return alert(error);
                })
        }
    }
}
</script>