<template>
    <tr>
        <td class="align-middle">
            <input type="checkbox" name="done" @change="updateStatus" :checked="status == 1" :disabled="status == 1">
        </td>

        <td class="align-middle">
            <span :class="{'atention': passed, 'done': status}">{{ task }}
            <span v-if="passed">(Atrasado)</span></span>
        </td>

        <td class="align-middle">
            <span v-if="status" :class="{'done': status}">Concluido</span>
            <span v-else :class="{'atention': passed}">{{ deadline }}</span>
        </td>

        <td class="text-right">
            <button v-if="!status" class="btn btn-info btn-sm text-light" @click="update">Editar</button>
            <button class="btn btn-danger btn-sm" @click="del">Delete</button>
        </td>
    </tr>
</template>

<script>

    export default {
        methods: {
            update() {
                this.$emit('update', this.id, this.task);
            },
            del() {
                this.$emit('delete', this.id);
            },
            updateStatus() {
                this.$emit('updateStatus', this.id);
            },
            isPassed() {            
                if (this.status) {
                    this.passed = false;
                } else {
                    const newDeadline = this.deadline.split(' ');

                    const oldDate = newDeadline[0].split('/').reverse().join('-');
                    const oldTime = newDeadline[1];

                    const dateFormated = `${oldDate}T${oldTime}`;

                    const date = new Date(dateFormated);
                    const dateNow = new Date();

                    if (date < dateNow) {
                        this.passed = true;
                    } else {
                        this.passed = false;
                    }
                }
            }
        },
        props: ['id', 'task', 'deadline', 'status'],
        data: () => {
            return ({
                passed: ''
            })
        },
        mounted() {

            this.isPassed();
            
            setInterval(() => {
                this.isPassed();
            }, 1000);

        }
    }
</script>


<style>
    .atention {
        color: red;
        font-weight: bold;
    }
    .done {
        color: #ddd;
        text-decoration: line-through;
    }
</style>