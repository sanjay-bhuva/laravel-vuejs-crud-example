<template>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                <div class="panel panel-default">
                    <div class="panel-heading">
                        <button @click="initAddProject()" class="btn btn-primary btn-xs pull-right">
                            + Add New Project
                        </button>
                        My Projects
                    </div>

                    <div class="panel-body">
                        <table class="table table-bordered table-striped table-responsive" v-if="projects.length > 0">
                            <tbody>
                                <tr>
                                    <th>
                                        No.
                                    </th>
                                    <th>
                                        Name
                                    </th>
                                    <th>
                                        Description
                                    </th>
                                    <th>
                                        Action
                                    </th>
                                </tr>
                                <tr v-for="(project, index) in projects">
                                    <td>{{ index + 1 }}</td>
                                    <td>
                                        {{ project.name }}
                                    </td>
                                    <td>
                                        {{ project.description }}
                                    </td>
                                    <td>
                                        <button @click="initUpdate(index)" class="btn btn-success btn-xs">Edit</button>
                                        <button @click="deleteProject(index)" class="btn btn-danger btn-xs">Delete</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <div class="modal fade" tabindex="-1" role="dialog" id="add_project_model">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                        <h4 class="modal-title">Add New Project</h4>
                    </div>
                    <div class="modal-body">

                        <div class="alert alert-danger" v-if="errors.length > 0">
                            <ul>
                                <li v-for="error in errors">{{ error }}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label for="name">Name:</label>
                            <input type="text" name="name" id="name" placeholder="Project Name" class="form-control"
                                   v-model="project.name">
                        </div>
                        <div class="form-group">
                            <label for="description">Description:</label>
                            <textarea name="description" id="description" cols="30" rows="5" class="form-control"
                                      placeholder="Project Description" v-model="project.description"></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                        <button type="button" @click="createProject" class="btn btn-primary">Submit</button>
                    </div>
                </div><!-- /.modal-content -->
            </div><!-- /.modal-dialog -->
        </div><!-- /.modal -->

        <div class="modal fade" tabindex="-1" role="dialog" id="update_project_model">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                                aria-hidden="true">&times;</span></button>
                        <h4 class="modal-title">Update Project</h4>
                    </div>
                    <div class="modal-body">

                        <div class="alert alert-danger" v-if="errors.length > 0">
                            <ul>
                                <li v-for="error in errors">{{ error }}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label>Name:</label>
                            <input type="text" placeholder="Project Name" class="form-control"
                                   v-model="update_project.name">
                        </div>
                        <div class="form-group">
                            <label for="description">Description:</label>
                            <textarea cols="30" rows="5" class="form-control"
                                      placeholder="Project Description" v-model="update_project.description"></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                        <button type="button" @click="updateProject" class="btn btn-primary">Submit</button>
                    </div>
                </div><!-- /.modal-content -->
            </div><!-- /.modal-dialog -->
        </div><!-- /.modal -->
    </div>
</template>

<script>
    export default {
        data() {
            return {
                projects: [],
                project: {
                    name: '',
                    description: ''
                },
                errors: [],
                update_project: {}
            }
        },
        mounted()
        {
            this.readProjects();
        },
        methods: {
            initAddProject()
            {
                this.errors = [];
                this.reset();
                $("#add_project_model").modal("show");
            },
            createProject()
            {
                axios.post('/project', {
                    name: this.project.name,
                    description: this.project.description,
                })
                        .then(response => {
                            this.reset();
                            this.projects.push(response.data.project);
                            $("#add_project_model").modal("hide");
                        })
                        .catch(error => {
                            this.errors = [];
                            if (error.response.data.errors.name) {
                                this.errors.push(error.response.data.errors.name[0]);
                            }

                            if (error.response.data.errors.description) {
                                this.errors.push(error.response.data.errors.description[0]);
                            }
                        });
            },
            reset()
            {
                this.project.name = '';
                this.project.description = '';
            },
            readProjects()
            {
                axios.get('/project')
                        .then(response => {
                            this.projects = response.data.projects;
                        });
            },
            initUpdate(index)
            {
                this.errors = [];
                $("#update_project_model").modal("show");
                this.update_project = this.projects[index];
            },
            updateProject()
            {
                axios.patch('/project/' + this.update_project.id, {
                    name: this.update_project.name,
                    description: this.update_project.description,
                })
                        .then(response => {
                            $("#update_project_model").modal("hide");
                        })
                        .catch(error => {
                            this.errors = [];
                            if (error.response.data.errors.name) {
                                this.errors.push(error.response.data.errors.name[0]);
                            }

                            if (error.response.data.errors.description) {
                                this.errors.push(error.response.data.errors.description[0]);
                            }
                        });
            },
            deleteProject(index)
            {
                let conf = confirm("Do you ready want to delete this project?");
                if (conf === true) {
                    axios.delete('/project/' + this.projects[index].id)
                    .then(response => {
                        this.projects.splice(index, 1);
                    })
                    .catch(error => {
                    });
                }
            }
        }
    }
</script>