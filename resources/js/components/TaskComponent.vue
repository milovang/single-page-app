<template>
    <div>
        <div v-if="!loading">
            <img :src="image" alt="loader" class="rounded mx-auto d-block">
        </div>
        <div v-else>
            <button @click="createModal" class="btn btn-primary btn-block">Add new task</button>
            <table class="table" v-if="tasks">
                <thead>
                <tr>
                    <th>Id</th>
                    <th>Name</th>
                    <th>Body</th>
                    <th></th>
                    <th></th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(task, index) in tasks">
                    <td>{{index + 1}}</td>
                    <td>{{task.name}}</td>
                    <td>{{task.body}}</td>
                    <td><button @click="updateModal(index)" class="btn btn-info">Edit</button></td>
                    <td><button @click="deleteTask(index)" class="btn btn-danger">Delete</button></td>
                </tr>
                </tbody>
            </table>

            <!-- Create Modal -->
            <div class="modal fade" id="create-modal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="exampleModalLabel">Create Modal</h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">

                            <div class="alert alert-danger" v-if="errors.length > 0">
                                <ul>
                                    <li v-for="error in errors">{{error}}</li>
                                </ul>
                            </div>

                            <div class="form-group">
                                <label for="name">Name</label>
                                <input v-model="task.name" type="text" id="name" class="form-control">
                            </div>

                            <div class="form-group">
                                <label for="body">Body</label>
                                <input v-model="task.body" type="text" id="body" class="form-control">
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            <button @click="createTask" type="button" class="btn btn-primary">Save changes</button>
                        </div>
                    </div>
                </div>
            </div>
            <!-- Update Modal -->
            <div class="modal fade" id="update-modal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="exampleModalLabel">Update Modal</h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">

                            <div class="alert alert-danger" v-if="errors.length > 0">
                                <ul>
                                    <li v-for="error in errors">{{error}}</li>
                                </ul>
                            </div>

                            <div class="form-group">
                                <label for="name">Name</label>
                                <input v-model="update_task.name" type="text" id="name" class="form-control">
                            </div>

                            <div class="form-group">
                                <label for="body">Body</label>
                                <input v-model="update_task.body" type="text" id="body" class="form-control">
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            <button @click="updateTask" type="button" class="btn btn-primary">Save changes</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {

        data(){
            return {
                task:{
                    name: '',
                    body: '',
                },
                tasks: [],
                errors: [],
                uri:'http://localhost:8000/tasks/',
                update_task: [],
                image: 'images/loader1.gif',
                loading: false,
            }
        },

        methods: {
            createModal(){
                $("#create-modal").modal("show");
            },
            updateModal(index){
                this.errors = [];
                $("#update-modal").modal("show");
                this.update_task = this.tasks[index];
            },

            createTask(){
                axios.post(this.uri,
                    {name: this.task.name, body: this.task.body}).then(response=>{
                        this.tasks.push(response.data.task);
                        this.resetData();
                        $("#create-modal").modal("hide");
                        toastr.success(response.data.message);

                }).catch(error=>{

                    this.errors = [];
                    if(error.response.data.errors.name){
                        this.errors.push(error.response.data.errors.name[0]);
                    }
                    if(error.response.data.errors.body){
                        this.errors.push(error.response.data.errors.body[0]);
                    }

                });
            },

            updateTask(){
                axios.patch(this.uri + this.update_task.id, {
                    name: this.update_task.name,
                    body: this.update_task.body,
                }).then(response =>{

                    $("#update-modal").modal("hide");
                    toastr.success(response.data.message);

                }).catch(error=>{

                this.errors = [];
                if(error.response.data.errors.name){
                    this.errors.push(error.response.data.errors.name[0]);
                }
                if(error.response.data.errors.body){
                    this.errors.push(error.response.data.errors.body[0]);
                }

                });
            },

            deleteTask(index){

                let confirmBox = confirm("Do you really want to delete?");

                if(confirmBox == true){
                    axios.delete(this.uri + this.tasks[index].id).then(response => {
                        this.$delete(this.tasks, index);
                        toastr.success(response.data.message);
                    }).catch(error=>{

                        console.log("Not deleted for some reason");

                    });
                }

            },

            loadTasks(){

                axios.get(this.uri).then(response=>{
                    this.tasks = response.data.tasks;
                    this.loading = true;
                });

            },

            resetData(){
                this.task.name = "";
                this.task.body = "";
            }
        },


        mounted() {
            this.loadTasks();
        },
    }
</script>
