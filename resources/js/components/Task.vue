<template>
    <div class="row my-3 align-items-center"> 
        <div class="col-md-8">              
            <span>{{ task }}</span>                
        </div>
        <div class="col-md-2">
            <button class="btn btn-info btn-block btn-sm text-light" @click="update">Editar</button>
        </div>
        <div class="col-md-2">
            <button class="btn btn-danger btn-block btn-sm" @click="del">Delete</button>
        </div>
    </div>
</template>

<script>
    export default {
        methods: {
            update() {
                this.$emit('update', this.id, this.task);
            },
            del() {
                this.$emit('delete', this.id);
            }
        },
        props: ['id', 'task']
    }
</script>