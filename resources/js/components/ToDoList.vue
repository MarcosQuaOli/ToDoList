<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h2>Nova tarefa</h2>
                        <form v-on:submit.prevent="insert">
                            <input type="hidden" name="_token" :value="csrf">
                            <div class="row">
                                <div class="col-md-8">
                                    <div class="form-group">
                                        <input class="form-control" type="text" name="task" placeholder="Digite a tarefa" v-model="name" required>
                                    </div>
                                </div>

                                <div class="col-md-4">
                                    <div class="form-group">
                                        <input id="deadline" class="form-control" type="datetime-local" name="deadline" :min="dateNow" required>
                                    </div>
                                </div>
                            </div>

                            <div>
                                <button class="btn btn-primary w-25" type="submit">Salvar</button>
                            </div>
                        </form>

                        <div v-if="edit">
                            <h2 class="mt-4">Editar tarefa</h2>
                            <form v-on:submit.prevent="updateForm">
                                <input type="hidden" name="_token" :value="csrf">
                                <div class="row">
                                    <div class="col-8">
                                        <div class="form-group">
                                            <input class="form-control" type="text" name="task" placeholder="Digite a tarefa" v-model="taskEdit">
                                        </div>
                                    </div>

                                    <div class="col-4">
                                        <button class="btn btn-primary btn-block" type="submit">Salvar</button>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </div>

                    <div class="card-body">
                        <div class="table-responsive-md">
                            <table class="table table-hover text-center">
                                <thead class="thead-dark">
                                    <tr>
                                        <th scope="col">Concluir</th>
                                        <th scope="col">Tarefa</th>
                                        <th scope="col">Prazo para conclusão</th>
                                        <th scope="col">Ações</th>
                                    </tr>
                                </thead>

                                <tbody>
                                    <Task v-for="task in tasks" v-bind="[task, deadline, status]" :key="task.id" @update="update" @updateStatus="updateStatus" @delete="del"></Task>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import Task from './Task.vue';

    function Tasks({ id, task, deadline, status }) {
        this.id = id;
        this.task = task;
        this.status = status;

        const date = formatDateBR(deadline);
        this.deadline = date;
    }

    function formatDateBR(deadline) {

        const date = new Date(deadline);
        const dateFormated = date.toLocaleDateString();
        const timeFormated = date.toLocaleTimeString().split(':');
        timeFormated.pop();
        const newTime = timeFormated.join(':');

        return `${dateFormated} ${newTime}`;

    }

    function formatDateMin() {

        const dateTarget = new Date();
        const dateFormated = dateTarget.toLocaleDateString().split('/').reverse().join('-');
        const timeFormated = dateTarget.toLocaleTimeString().split(':');
        timeFormated.pop();
        const newTime = timeFormated.join(':');

        return `${dateFormated}T${newTime}`;

    }

    export default {
        components: { Task },
        data: () => {
            return ({
                csrf: document.querySelector('meta[name="csrf-token"]').getAttribute('content'),
                name: '',
                tasks: [],
                edit: false,
                taskEdit: '',
                idEdit: '',
                deadline: '',
                status: '',
                dateNow: formatDateMin()
            })
        },
        methods: {
            async insert() {
                const deadline = document.getElementById('deadline');
                
                window.axios.post('/api/tasks', {task: this.name, deadline: deadline.value})
                    .then(resp => {
                        this.tasks.push(new Tasks(resp.data));
                        this.name = '';
                        deadline.value = '';
                    });
            },
            async read() {
                window.axios.get('/api/tasks')
                    .then(resp => {
                        resp.data.forEach(task => this.tasks.push(new Tasks(task)));    
                    });
            },
            update(id, task) {
                this.taskEdit = task;
                this.idEdit = id;
                this.edit = true;
            },
            updateForm() {
                window.axios.put(`/api/tasks/${this.idEdit}`, {task: this.taskEdit});
                this.edit = false;

                let index = this.tasks.findIndex(task => task.id === this.idEdit);
                this.tasks[index].task = this.taskEdit;
            },
            updateStatus(id) {
                window.axios.put(`/api/tasks/status/${id}`);

                let index = this.tasks.findIndex(task => task.id === id);
                this.tasks[index].status = true;
            },
            async del(id) {
                window.axios.delete(`/api/tasks/${id}`);
                let index = this.tasks.findIndex(task => task.id === id);
                this.tasks.splice(index, 1);
            }
        },
        mounted() {
            this.read();
        }
    }
</script>