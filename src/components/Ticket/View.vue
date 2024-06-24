<template>
    <div class="d-flex flex-column align-items-center gap-4" style="padding-left:5%; padding-right: 5%">

        <div class="d-flex align-items-center w-100">
            <div class="h4 d-flex pe-2 m-0">#{{ ticket.ID_ticket }}</div>
            <div class="h4 d-flex flex-grow-1 m-0">- {{ ticket.NAME }}</div>

            <div class="d-flex flex-column w-auto">
                <label>Encargado:</label>

                <p v-if="viewRoleBased() == 0" class="form-control m-0">AAAAA</p>

                <button v-if="viewRoleBased() == 1" class="btn btn-secondary">AAAAA</button>

                <select v-model="newAssinedTo" v-if="viewRoleBased() == 2"
                    class="text-reset btn btn-outline-secondary text-start" style="background-color: #212529">
                    <option :value="null" selected>Sin asignar</option>
                    <option v-for="user in assignableUsers" 
                    style="background-color: #212529;" 
                    :value="user.ID_usuario"
                    :selected="ticket.IDencargado == user.ID_usuario">
                        {{ user.nomusua }}
                    </option>
                </select>
            </div>
        </div>

        <div class="d-flex justify-content-center w-100 gap-4">
            <div class="w-50">
                <label>Creado por:</label>
                <p class="form-control">{{ findUserById(ticket.IDusuario).nomusua }}</p>
            </div>
            <div class="w-50">
                <label>Fecha de publicación:</label>
                <p class="form-control">{{ new Date(ticket.Fecha).toLocaleDateString() }}</p>
            </div>
        </div>

        <div class="d-flex justify-content-center w-100 gap-4">
            <div class="w-50">
                <label>Prioridad</label>
                <select v-model="newPriority"
                class="text-reset btn btn-outline-secondary text-start w-100"
                style="background-color: #212529">
                    <option v-for="prioridad in priorityList"
                    style="background-color: #212529;"
                    :value="prioridad.ID_prio"
                    :selected="ticket.IDprio == prioridad.ID_prio">
                        {{ prioridad.tipoprio }}</option>
                </select>
            </div>
            <div class="w-50">
                <label>Estatus</label>
                <select v-model="newStatus"
                class="text-reset btn btn-outline-secondary text-start w-100"
                style="background-color: #212529">
                    <option v-for="status in statusList"
                    style="background-color: #212529;"
                    :value="status.ID_status"
                    :selected="ticket.IDstatus == status.ID_status">
                        {{ status.tipostatus }}</option>
                </select>
            </div>
        </div>

        <div class="w-100">
            <label>Descripción</label>
            <textarea type="text"
            style="background-color: transparent;"
            class="form-control"
            placeholder="Descripción del problema"
            rows="8"
            disabled
            :value="ticket.Descripcion"></textarea>
        </div>

        <button v-if="newAssinedTo != ticket.IDencargado || newPriority != this.ticket.IDprio || newStatus != this.ticket.IDstatus"
        type="button"
        class="btn btn-success mb-3 w-100"
        @click.prevent="updateTicket()">
            Guardar Cambios
        </button>
    </div>

    <div class="border m-4"></div>

    <div style="padding-left:5%; padding-right: 5%">
        <div class="d-flex flex-column align-items-center gap-4 border rounded-2 p-3">
            <div v-if="comments.length != 0" class="h5 d-flex w-100 bold m-0">Comentarios</div>
            <div class="w-100">
                <label>Agrega un comentario</label>
                <textarea v-model="newComment" type="text" class="form-control" placeholder="Comentario"
                    rows="4"></textarea>
                <div class="d-flex flex-row-reverse">
                    <button type="button" class="btn btn-success mb-3 w-25" @click.prevent="postComment()">
                        Comentar
                    </button>
                </div>
            </div>

            <div v-for="comment in comments" class="w-100">
                <div class="d-flex flex-column w-100 h-100">
                    <div class="d-flex border">
                        <div class="d-flex flex-column w-25 justify-content-evenly">
                            <p class="m-0 text-center fs-5">{{ findUserById(comment.IDusuario).nomusua }}</p>
                            <p class="m-0 text-center fs-6 border-bottom border-dark-subtle">{{ roles[Number(findUserById(comment.IDusuario).rol)] }}</p>
                            <p class="m-0 text-center fs-6">{{ new Date(comment.Fecha).toLocaleDateString() }}</p>
                            <p class="m-0 text-center fs-6">{{ new Date(comment.Fecha).toLocaleTimeString() }}</p>
                        </div>
                        <div class="w-75">
                            <textarea type="text"
                            style="background-color: transparent;"
                            class="form-control" rows="4"
                            :value="comment.Comentario"
                            disabled></textarea>
                        </div>
                    </div>
                    <div class="d-flex justify-content-end">
                        <button v-if="loggedUser.id == comment.IDusuario"
                        class="btn btn-danger h-auto"
                        @click.prevent="deleteComment(comment.ID_coment)">
                            Eliminar
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <div class="w-100" style="height: 2em;"></div>
    </div>
</template>

<script>
export default {
    props: ["loggedUser", "ticket", "statusList", "priorityList", "userList", "assignableUsers"],
    data() {
        return {
            newAssinedTo: this.ticket.IDencargado,
            newPriority: this.ticket.IDprio,
            newStatus: this.ticket.IDstatus,
            comments: [],
            newComment: '',
            roles: ['Usuario', 'Tecnico', 'Administrador']
        }
    },
    mounted() {
        this.getComments();
    },
    methods: {
        viewRoleBased() {
            switch (this.loggedUser.role) {
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
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id);
        },
        async postComment() {
            await axios.post(`${process.env.VUE_APP_BACKENDURL}/comments/`, { ticketid: this.ticket.ID_ticket, text: this.newComment, userid: this.loggedUser.id, date: new Date().toISOString().replace('T', ' ').replace('Z', ' ') }).then(
                (response) => {
                    this.newComment = '';
                    this.getComments();
                }).catch((error) => {
                    return alert(`Ha surgido un error al realizar el comentario`);
                })
        },
        getComments() {
            axios.get(`${process.env.VUE_APP_BACKENDURL}/comments/?id=${this.ticket.ID_ticket}`).then(
                (response) => {
                    this.comments = response.data;
                }
            )
        },
        async deleteComment(commentId) {
            await axios.delete(`${process.env.VUE_APP_BACKENDURL}/comments/?id=${commentId}`).then(
                async (response) => {
                    await this.getComments();
                }).catch((error) => {
                    return alert('Ha surgido un error al eliminar el comentario');
                })
        },
        async updateTicket() {
            await axios.put(`${process.env.VUE_APP_BACKENDURL}/tickets/?id=${this.ticket.ID_ticket}`, { idencargado: this.newAssinedTo, idstatus: this.newStatus, idprioridad: this.newPriority }).then(
                (response) => {
                    alert(`Ticket actualizado con éxito`);
                    this.$parent.loadData(1);
                }).catch((error) => {
                    return alert('Ha surgido un error al actualizar el ticket');
                })
        }
    }
}
</script>