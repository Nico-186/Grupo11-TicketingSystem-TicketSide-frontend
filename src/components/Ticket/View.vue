<template>
    <div class="container-fluid h-100 v-100 py-4" style="padding-left:5%; padding-right: 15%">
        <div class="d-flex flex-column align-items-center gap-3">

            <div class="d-flex align-items-center w-100">
                <div class="h4 d-flex pe-2 m-0">#{{ ticket.ID_ticket }}</div>
                <div class="h4 d-flex flex-grow-1 m-0">- {{ ticket.NAME }}</div>

                <div class="d-flex form-group flex-column w-auto">
                    <label>Encargado:</label>

                    <p v-if="viewRoleBased() == 0" class="form-control m-0">AAAAA</p>

                    <button v-if="viewRoleBased() == 1" class="btn btn-secondary">AAAAA</button>

                    <select v-model="newAssinedTo" v-if="viewRoleBased() == 2"
                        class="text-reset btn btn-outline-secondary text-start" style="background-color: #212529">
                        <option :value="null" selected>Sin asignar</option>
                        <option v-for="user in assignableUsers" style="background-color: #212529;"
                            :value="user.ID_usuario" :selected="ticket.IDencargado == user.ID_usuario">
                            {{user.nomusua}}
                        </option>
                    </select>

                </div>
            </div>

            <div class="d-flex justify-content-center w-100 gap-3">
                <div class="form-group w-50">
                    <label>Creado por:</label>
                    <p class="form-control">{{ findUserById(ticket.IDusuario).nomusua }}</p>
                </div>
                <div class="form-group w-50">
                    <label>Fecha de creación:</label>
                    <p class="form-control">{{ new Date(ticket.Fecha).toLocaleDateString() }}</p>
                </div>
            </div>

            <div class="d-flex justify-content-center w-100 gap-3">
                <div class="w-50">
                    <label>Prioridad del ticket</label>
                    <select v-model="newPriority" class="text-reset btn btn-outline-secondary text-start w-100"
                        style="background-color: #212529">
                        <option v-for="prioridad in prioridades" style="background-color: #212529;"
                            :value="prioridad.ID_prio" :selected="ticket.IDprio == prioridad.ID_prio">
                            {{ prioridad.tipoprio }}</option>
                    </select>
                </div>
                <div class="w-50">
                    <label>Estatus del ticket</label>
                    <select v-model="newStatus" class="text-reset btn btn-outline-secondary text-start w-100"
                        style="background-color: #212529">
                        <option v-for="stat in status" style="background-color: #212529;" :value="stat.ID_status"
                            :selected="ticket.IDstatus == stat.ID_status">
                            {{ stat.tipostatus }}</option>
                    </select>
                </div>
            </div>

            <div class="form-group w-100">
                <label>Descripción</label>
                <textarea type="text" style="background-color: transparent;" class="form-control"
                    placeholder="Descripción del problema" rows="8" disabled :value="ticket.Descripcion"></textarea>
            </div>

            <button
                v-if="newAssinedTo != ticket.IDencargado || newPriority != this.ticket.IDprio || newStatus != this.ticket.IDstatus"
                type="button" class="btn btn-success mb-3 w-100">
                Guardar Cambios
            </button>

            <div class="form-group w-100">
                <label>Dejar comentario</label>
                <textarea v-model="newComment" type="text" class="form-control" placeholder="Comentario"
                    rows="4"></textarea>
                <div class="d-flex flex-row-reverse">
                    <button type="button" class="btn btn-success mb-3 w-25" @click.prevent="postComment()">
                        Comentar
                    </button>
                </div>
            </div>

            <div v-if="comments.length != 0" class="h5 d-flex w-100 bold m-0">Comentarios</div>
            <div v-for="comment in comments" class="w-100">
                <div class="d-flex flex-column w-100 h-100">
                    <div class="d-flex border">
                        <div class="d-flex flex-column w-25 justify-content-evenly">
                            <p class="m-0 text-center fs-5">{{findUserById(comment.IDusuario).nomusua}}</p>
                            <p class="m-0 text-center fs-6 border-bottom border-dark-subtle">
                                {{roles[Number(findUserById(comment.IDusuario).rol)]}}</p>
                            <p class="m-0 text-center fs-6">{{new Date(comment.Fecha).toLocaleDateString()}}</p>
                            <p class="m-0 text-center fs-6">{{new Date(comment.Fecha).toLocaleTimeString()}}</p>
                        </div>
                        <div class="form-group w-75">
                            <textarea type="text" style="background-color: transparent;" class="form-control" rows="4"
                                :value="comment.Comentario" disabled></textarea>
                            <div class="d-flex flex-row-reverse">
                            </div>
                        </div>
                    </div>
                    <div class="d-flex justify-content-end">
                        <button v-if="loggedUser.id == comment.IDusuario" class="btn btn-danger h-auto" @click.prevent="deleteComment(comment.ID_coment)">Eliminar</button>
                    </div>
                </div>
            </div>
            <div class="w-100" style="height: 2em;"></div>

        </div>
    </div>
</template>

<script>
import moment from 'moment';

export default {
    props: ["ticket","loggedUser","status","prioridades","userList","assignableUsers","isOpen"],
    data() {
        return {
            isMiau: false,
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
            return this.status.find((obj) => obj.ID_status == id).tipostatus;
        },
        findPriorityById(id) {
            return this.prioridades.find((obj) => obj.ID_prio == id).tipoprio;
        },
        findUserById(id) {
            return this.userList.find((obj) => obj.ID_usuario == id);
        },
        async postComment() {
            await axios.post(`http://localhost:3000/comments/`, { ticketid: this.ticket.ID_ticket, text: this.newComment, userid: this.loggedUser.id, date: moment().format().slice(0, 19).replace('T', ' ')}).then(
                    (response) => {
                        this.newComment = '';
                        this.getComments();
                    }).catch((error) =>{
                        return alert(`Ha surgido un error al realizar el comentario`);
                    })
        },
        getComments() {
            axios.get(`http://localhost:3000/comments/?id=${this.ticket.ID_ticket}`).then(
                (response) => {
                    if (JSON.stringify(response.data) != JSON.stringify([])) {
                        this.comments = response.data;
                    }
                }
            )
        },
        async deleteComment(commentId) {
            await axios.delete(`http://localhost:3000/comments/?id=${commentId}`).then(
                (response) => {
                    this.getComments();
                }).catch((error) => {
                    return alert(error);
                })
        }
    }
}
</script>