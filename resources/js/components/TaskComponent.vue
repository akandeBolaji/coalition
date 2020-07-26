<template>
    <div class="row">
        <div class="col-12">
            <h2>Add task</h2>
            <form @submit.prevent="add">
                <div v-if="errors" class="alert alert-danger" role="alert">
                    <h5>Please, fix validation errors</h5>
                    <ul v-for="field in errors">
                        <li>{{ field[0] }}</li>
                    </ul>
                </div>
                <div class="form-group">
                    <label for="name">Name</label>
                    <input type="text" class="form-control" id="name" name="name" v-model="task.name">
                </div>
                <div class="form-group">
                    <label for="quantity">Priority</label>
                    <input type="number" class="form-control" id="priority" name="priority" v-model="task.priority">
                </div>
                <button type="submit" class="btn btn-primary">Submit</button>
            </form>
        </div>
        <div class="col-12 mt-5">
            <template v-if="tasks.length">
                <h2>All Tasks</h2>
                <table class="table">
                    <thead>
                        <tr>
                            <td scope="col">Task Name</td>
                            <th scope="col">Priority</th>
                            <th scope="col">Delete</th>
                        </tr>
                    </thead>
                    <tbody>
                        <draggable v-model="tasks" group="people" @start="drag=true" @end="drag=false">
                            <tr v-for="task, i in tasks" :key="task.id">
                                <td class="editable" ref="name" contenteditable @blur="update(i, task)">{{ task.name }}</td>
                                <td class="editable" ref="priority" contenteditable @blur="update(i, task)">{{ task.priority }}</td>
                                <td>
                                    <button @click="remove(i, task)" type="button" class="btn btn-danger">Delete</button>
                                </td>
                            </tr>
                        </draggable>
                    </tbody>
                </table>
            </template>
            <div v-else>No tasks</div>
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable'
    export default {
        data () {
            return {
                tasks: [],
                errors: null,
                task: {
                    name: null,
                    priority: null,
                }
            }
        },

        components: {
            draggable,
        },

        mounted() {
            this.getTasks();
        },

        methods: {
            getTasks() {
                axios.get('/api/tasks').then((response) => {
                    this.tasks = response.data
                })
            },
            add() {
                axios.post('/api/tasks', this.task).then((response) => {
                    this.tasks.unshift(response.data)
                    this.errors = null

                    this.task = {
                        name: null,
                        priority: null,
                    }
                   this.getTasks();
                }).catch(error => {
                    this.errors = error.response.data.errors
                })
            },
            update(id, task) {
                let updatedTask = {
                    name: this.$refs.name[id].textContent,
                    priority: this.$refs.priority[id].textContent,
                };

                console.log([this.tasks[id], updatedTask])

                if (this.isEqual(this.tasks[id], updatedTask)) {
                    return
                }

                axios.patch(`/api/tasks/${id}`, updatedTask).then((response) => {
                    this.$delete(this.tasks, id)
                    this.tasks.unshift(response.data)

                    this.errors = null
                    this.getTasks();
                }).catch(error => {
                    this.errors = error.response.data.errors
                })
            },
            remove(id, task) {
                axios.delete(`api/tasks/${id}`).then(response => {
                    this.tasks.splice(id, 1)
                    this.errors = null
                    this.getTasks();
                }).catch(error => {
                    this.errors = error.response.data.errors
                })
            },
            isEqual(task, updatedTask) {
                if (task.name != updatedTask.name) {
                    return false;
                }
                return true;
            }
        }
    }
</script>
