<template>
    <section>

        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.name" placeholder="Search"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCategories">Search</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">Add</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <el-table :data="categories" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
                  style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="Name" width="180" sortable>
            </el-table-column>
            <el-table-column prop="description" label="Description" width="400" sortable>
            </el-table-column>
            <el-table-column prop="sort" label="Sort Order" width="150" :formatter="formatSex" sortable>
            </el-table-column>
            <el-table-column prop="parent_category_id" label="Parent Category" width="200" sortable>
            </el-table-column>
            <el-table-column label="Actions" width="300">
                <template scope="scope">
                    <el-button size="small" type="warning" @click="handleEdit(scope.$index, scope.row)">Activate
                    </el-button>
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">Edit</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">Delete</el-button>
                </template>
            </el-table-column>
        </el-table>

        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">Delete Marked</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="20"
                           :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <el-dialog title="Edit Category" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="Name" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="Parent" prop="parent_category_id">
                    <el-select v-model="editForm.parent_category_id" value="-1" clearable>
                        <el-option
                                v-for="option in categories"
                                :key="option.id"
                                :label="option.name"
                                :value="option.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="Visible">
                    <el-radio-group v-model="editForm.status">
                        <el-radio class="radio" :label="1">Enabled</el-radio>
                        <el-radio class="radio" :label="0">Disabled</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="Sort Order">
                    <el-input-number v-model="editForm.sort" :min="0" :max="200"></el-input-number>
                </el-form-item>
                <el-form-item label="Address">
                    <el-input type="textarea" v-model="editForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">Cancel</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">Save</el-button>
            </div>
        </el-dialog>


        <el-dialog title="Add Category" v-model="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="Name" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="Parent" prop="parent_category_id">
                    <el-select v-model="addForm.parent_category_id" value="-1" clearable>
                        <el-option
                                v-for="option in categories"
                                :key="option.id"
                                :label="option.name"
                                :value="option.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="Visible" prop="status">
                    <el-radio-group v-model="addForm.status">
                        <el-radio class="radio" :label="1">Enabled</el-radio>
                        <el-radio class="radio" :label="0">Disabled</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="Sort Order" prop="sort">
                    <el-input-number v-model="addForm.sort" :min="0" :max="200"></el-input-number>
                </el-form-item>
                <el-form-item label="Description" prop="description">
                    <el-input type="textarea" v-model="addForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">Cancel</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">Save</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import util from '../../common/js/util'
    //import NProgress from 'nprogress'
    import {
        getUserListPage,
        removeUser,
        batchRemoveUser,
        editUser,
        addUser,
        getCategories,
        addCategory, editCategory
    } from '../../api/api';

    export default {
        components: {},
        name: 'categories',
        data() {
            return {
                filters: {
                    name: ''
                },
                categories: [],
                total: 0,
                page: 1,
                listLoading: false,
                sels: [],

                editFormVisible: false,
                editLoading: false,
                editFormRules: {
                    name: [
                        {required: true, message: 'Required', trigger: 'blur'}
                    ]
                },
                //
                editForm: {
                    id: 0,
                    parent_category_id: 0,
                    name: '',
                    description: '',
                    sort: 0,
                    image: '',
                    thumbnail: '',
                    status: 0
                },

                addFormVisible: false,
                addLoading: false,
                addFormRules: {
                    name: [
                        {required: true, message: 'Required', trigger: 'blur'}
                    ]
                },
                //
                addForm: {
                    parent_category_id: 0,
                    name: '',
                    description: '',
                    sort: 0,
                    image: '',
                    thumbnail: '',
                    status: 0
                }

            }
        },
        methods: {
            //
            formatSex: function (row, column) {
                return row.sex == 1 ? 'Male' : row.sex == 0 ? 'Female' : 'ddd';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getCategories();
            },
            //
            getCategories() {
                let para = {
                    page: this.page,
                    name: this.filters.name
                };
                this.listLoading = true;
                //NProgress.start();
                getCategories(para).then((res) => {
                    //this.total = res.data.total;
                    console.log(res);
                    this.categories = res.data.data;
                    this.listLoading = false;
                    //NProgress.done();
                });
            },
            //
            handleDel: function (index, row) {
                this.$confirm('Delete USer?', 'Delete', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = {id: row.id};
                    removeUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: 'Deleted',
                            type: 'success'
                        });
                        this.getCategories();
                    });
                }).catch(() => {

                });
            },
            //
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //
            handleAdd: function () {
                this.addFormVisible = true;
                this.addForm = {
                    parent_category_id: 0,
                    name: '',
                    description: '',
                    sort: 0,
                    image: '',
                    thumbnail: '',
                    status: 0
                }
            },
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('Update？', 'Edit', {}).then(() => {
                            this.editLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            editCategory(para).then((res) => {
                                this.editLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: 'Updated',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.editFormVisible = false;
                                this.getCategories();
                            });
                        });
                    }
                });
            },
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('Save Category？', 'Save', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.addForm);
                            //console.log(para);
                            addCategory(para).then((res) => {
                                this.addLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: 'Saved.',
                                    type: 'success'
                                });
                                this.$refs['addForm'].resetFields();
                                this.addFormVisible = false;
                                this.getCategories();
                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('Delete  selected？', 'Delete', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = {ids: ids};
                    batchRemoveUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: 'Deleted',
                            type: 'success'
                        });
                        this.getCategories();
                    });
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getCategories();
        }
    }

</script>

<style scoped>

</style>