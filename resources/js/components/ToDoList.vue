<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-10">
                <div class="card">
                    <div class="card-header">
                        <h2>Nova tarefa</h2>
                        <form v-on:submit.prevent="insert">
                            <input type="hidden" name="_token" :value="csrf">
                            <div class="row">
                                <div class="col-8">
                                    <div class="form-group">
                                        <input class="form-control" type="text" name="task" placeholder="Digite a tarefa" v-model="name">
                                    </div>
                                </div>

                                <div class="col-4">
                                    <button class="btn btn-primary btn-block" type="submit">Salvar</button>
                                </div>
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
                        <Task v-for="task in tasks" v-bind="task" :key="task.id" @update="update" @delete="del"></Task>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import Task from './Task.vue';

    function Tasks({ id, task}) {
        this.id = id;
        this.task = task;
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
                idEdit: ''
            })
        },
        methods: {
            async insert() {
                window.axios.post('/api/tasks', {task: this.name})
                    .then(resp => {
                        this.tasks.push(new Tasks(resp.data));
                        this.name = '';
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