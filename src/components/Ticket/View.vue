<template>
    <div class="d-flex flex-column align-items-center gap-4" style="padding-left:5%; padding-right: 5%">

        <div class="d-flex align-items-center w-100">
            <div class="h4 d-flex pe-2 m-0">#{{ ticket.ID_ticket }}</div>
            <div class="h4 d-flex flex-grow-1 m-0">- {{ ticket.NAME }}</div>

            <div class="d-flex flex-column w-auto">
                <label>Encargado:</label>
                <p v-if="interactPrivileges == 0" class="form-control m-0">{{ ticket.IDencargado != null ? findUserById(ticket.IDencargado).nomusua : 'Sin asignar' }}</p>

                <button v-if="interactPrivileges == 1" class="btn" :class="newAssinedTo == this.loggedUser.id ? 'btn-outline-success' : 'btn-success'" @click="changeAssignedTo()">Adjudicar Ticket</button>

                <button v-if="interactPrivileges == 2" class="btn" :class="newAssinedTo == null ? 'btn-outline-danger' : 'btn-danger'" @click="changeAssignedTo()">Abandonar Ticket</button>

                <select v-if="this.loggedUser.role == 2"
                v-model="newAssinedTo"
                class="text-reset btn btn-outline-secondary text-start" 
                style="background-color: #212529">
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
                
                <p v-if="interactPrivileges == 0 || interactPrivileges == 1" class="form-control m-0">{{ findPriorityById(ticket.IDprio).tipoprio }}</p>

                <select v-if="this.loggedUser.role == 2 || interactPrivileges == 2" 
                v-model="newPriority"
                class="text-reset btn btn-outline-secondary text-start w-100"
                style="background-color: #212529">
                    <option v-for="prioridad in priorityList"
                    style="background-color: #212529;"
                    :value="prioridad.ID_prio"
                    :selected="ticket.IDprio == prioridad.ID_prio">
                        {{ prioridad.tipoprio }}</option>
                </select>

            </div>
            <div class="d-flex justify-content-center w-50 gap-4">
                <div class="w-50">
                    <label>Estatus</label>
                    <p v-if="interactPrivileges == 0 || interactPrivileges == 1" class="form-control m-0">{{ findStatusById(ticket.IDstatus).tipostatus }}</p>
                    <select v-if="this.loggedUser.role == 2 || interactPrivileges == 2"
                    v-model="newStatus"
                    class="text-reset btn btn-outline-secondary text-start w-100"
                    style="background-color: #212529">
                        <option v-for="status in statusList"
                        style="background-color: #212529;"
                        :value="status.ID_status"
                        :selected="ticket.IDstatus == status.ID_status">
                            {{ status.tipostatus }}</option>
                    </select>
                </div>
                <div class="w-50">
                    <label>Tiempo transcurrido en {{ findStatusById(newStatus).tipostatus }}</label>
                    <p class="form-control m-0">{{ findStatusById(this.newStatus).statusFinal.data[0] == 1 ? 'Ticket Finalizado' : getElapsedTime() }}</p>
                </div>
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
            <div v-for="comment in comments" class="w-100">
                <div class="d-flex flex-column w-100 h-100">
                    <div class="d-flex border rounded-2">
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
                    <div class="d-flex">
                        <div class="flex-grow-1">
                            <button v-if="loggedUser.role != 0"
                            :class="comment.final.data[0] == 1 ? 'btn-outline-secondary' : 'btn-secondary'"
                            class="btn h-auto"
                            @click.prevent="updateComment(comment.ID_coment,0,comment.final.data[0] == 1 ? 0 : 1)">
                                Final
                            </button>
                        </div>
                        <button v-if="loggedUser.id == comment.IDusuario && loggedUser.role != 0"
                        :class="comment.interno.data[0] == 1 ? 'btn-outline-secondary' : 'btn-secondary'"
                        class="btn h-auto"
                        @click.prevent="updateComment(comment.ID_coment,comment.interno.data[0] == 1 ? 0 : 1,0)">
                            Interno
                        </button>
                        <button v-if="loggedUser.id == comment.IDusuario"
                        class="btn btn-danger h-auto"
                        @click.prevent="deleteComment(comment.ID_coment)">
                            Eliminar
                        </button>
                    </div>
                </div>
            </div>
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
            allComments: [],
            newComment: '',
            roles: ['Usuario', 'Tecnico', 'Administrador'],
            interactPrivileges: -1,
            time: ''
        }
    },
    async mounted() {
        await this.getTime();
        await this.getComments();
        this.getCommentsToShow();
        if (this.loggedUser.role != 2){
            this.getInteractPrivileges();
        }
    },
    methods: {
        changeAssignedTo(){
            if (this.newAssinedTo == this.loggedUser.id){
                this.newAssinedTo = null;
            } else {
                this.newAssinedTo = this.loggedUser.id;
            }
        },
        getElapsedTime() {

            let elapsedTimeMS = new Date() - new Date(this.time.inicio);

            let days = Math.abs(elapsedTimeMS/(1000*60*60*24))
            let hours = Math.abs((days - Math.floor(days)) * 24);
            let minutes = Math.abs((hours - Math.floor(hours)) * 60);

            return `${Math.floor(days)} días, ${Math.floor(hours)}:${this.singleDigitFix(Math.floor(minutes))} hrs`;
        },
        singleDigitFix(digit) {
            if(digit.toString().length == 1) {
                return '0' + digit.toString();
            } else {
                return digit;
            }
        },
        getCommentsToShow() {
            this.comments = [];
            let finalComment = this.allComments.find((obj) => obj.final.data[0] == 1)
            if (finalComment != undefined) {
                this.allComments.splice(this.allComments.indexOf(finalComment),1);
                this.comments.push(finalComment);
            }

            if (this.loggedUser.role == 0) {
                this.comments.push(...this.allComments.filter(function (comment) {
                    return comment.interno.data[0] != 1;
                }));
            } else {
                this.comments.push(...this.allComments);
            }
        },
        getInteractPrivileges() {
            switch (this.loggedUser.role) {
                case '0':
                    return this.interactPrivileges = 0;
                case '1':
                    if (this.ticket.IDencargado == null) {
                        return this.interactPrivileges = 1;
                    } else {
                        if (this.ticket.IDencargado == this.loggedUser.id) {
                            return this.interactPrivileges = 2;
                        } else {
                            return this.interactPrivileges = 0;
                        }
                    }
            }
        },
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id);
        },
        findPriorityById(id) {
            return this.priorityList.find((obj) => obj.ID_prio == id)
        },
        findStatusById(id) {
            return this.statusList.find((obj) => obj.ID_status == id)
        },
        async postComment() {
            await axios.post(`${process.env.VUE_APP_BACKENDURL_TICKET}/comments/`, { ticketid: this.ticket.ID_ticket, text: this.newComment, userid: this.loggedUser.id, date: new Date().toISOString().replace('T', ' ').replace('Z', ' ') }).then(
                async (response) => {
                    this.newComment = '';
                    await this.getComments();
                    this.getCommentsToShow();
                }).catch((error) => {
                    return alert(`Ha surgido un error al realizar el comentario`);
                })
        },
        async getTime() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/time/?idstatus=${this.newStatus}&idticket=${this.ticket.ID_ticket}`).then(
                (response) => {
                    this.time = response.data[0][0];
                }
            )
        },
        async getComments() {
            await axios.get(`${process.env.VUE_APP_BACKENDURL_TICKET}/comments/?id=${this.ticket.ID_ticket}`).then(
                (response) => {
                    this.allComments = response.data[0];
                }
            )
        },
        async deleteComment(commentId) {
            await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/comments/delete/?id=${commentId}`).then(
                async (response) => {
                    await this.getComments();
                    this.getCommentsToShow();
                }).catch((error) => {
                    return alert('Ha surgido un error al eliminar el comentario');
                })
        },
        async updateComment(commentId, isInternal, isFinal) {
            await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/comments/?id=${commentId}`, { interno: isInternal, final: isFinal}).then(
                async (response) => {
                    await this.getComments();
                    this.getCommentsToShow();
                }).catch((error) => {
                    return alert('Ha surgido un error al eliminar el comentario');
                })
        },
        async updateTicket() {
            if (this.newStatus != this.ticket.IDstatus){
                await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/time/?id=${this.time.ID_tiempo}`, { end: new Date().toISOString().replace('T', ' ').replace('Z', ' ') })
                if (this.findStatusById(this.newStatus).statusFinal.data[0] != 1){
                    await axios.post(`${process.env.VUE_APP_BACKENDURL_TICKET}/time/`, { status: this.newStatus, start: new Date().toISOString().replace('T', ' ').replace('Z', ' '), ticket: this.time.ID_ticket })
                }
            }
            await axios.put(`${process.env.VUE_APP_BACKENDURL_TICKET}/tickets/?id=${this.ticket.ID_ticket}`, { idencargado: this.newAssinedTo, idstatus: this.newStatus, idprioridad: this.newPriority }).then(
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