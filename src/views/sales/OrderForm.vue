<template>
    <section>
        <el-row :gutter="20">
            <el-col :span="8">
                <el-card class="card-details">
                    <h3>Order Details</h3>
                </el-card>
            </el-col>
            <el-col :span="8">
                <el-card class="card-details">
                    <h3>Customer Details</h3>
                </el-card>
            </el-col>
            <el-col :span="8">
                <el-card class="card-details">
                    <h3>Payment Details</h3>
                </el-card>
            </el-col>
        </el-row>
        <el-row >
            <el-col :span="24">
                <el-card>
                        <el-table :data="orderProducts" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
                            <el-table-column type="selection" width="55">
                            </el-table-column>
                            <el-table-column type="index" width="60">
                            </el-table-column>
                            <el-table-column prop="name" label="Name" width="180" sortable>
                            </el-table-column>
                            <el-table-column prop="quantity" label="Quantity" width="250" sortable>
                            </el-table-column>
                            <el-table-column prop="ammount" label="Ammount" width="200" sortable>
                            </el-table-column>
                            <el-table-column label="Actions" width="300">
                                <template scope="scope">
                                    <el-button size="small" type="warning" @click="handleEdit(scope.$index, scope.row)">Activate</el-button>
                                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">Edit</el-button>
                                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">Delete</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                </el-card>
            </el-col>
        </el-row>
        <el-dialog title="Edit Order Product" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="Name" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>

                <el-form-item label="Quantity">
                    <el-input-number type="textarea" v-model="editForm.quantity"></el-input-number>
                </el-form-item>
                <el-form-item label="Ammount">
                    <el-input type="textarea" v-model="editForm.ammount"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">Cancel</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">Save</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import util from '../../common/js/util'
    //import NProgress from 'nprogress'
    import {removeUser, batchRemoveUser, getOrdersProducts,editOrderProducts} from '../../api/api';

    export default {
        name: 'order-products',
        data() {
            return {
                name: '',
                quantity: '1',
                ammout:'',
                filters: {
                    name: ''
                },
                orderProducts: [],
                total: 0,
                page: 1,
                listLoading: false,
                sels: [],

                editFormVisible: false,
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: 'Required', trigger: 'blur' }
                    ]
                },
                //
                editForm: {
                    id: 0,
                    name: '',
                    quantity: '1',
                    ammout:'',
                },

                addFormVisible: false,
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: 'Required', trigger: 'blur' }
                    ]
                },

            }
        },
        methods: {
            //
            formatSex: function (row, column) {
                return row.sex == 1 ? 'Male' : row.sex == 0 ? 'Female' : 'ddd';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getOrdersProducts();
            },
            //
            handleDel: function (index, row) {
                this.$confirm('Delete Manufacturer?', 'Delete', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                //NProgress.start();
                let para = { id: row.id };
                removeUser(para).then((res) => {
                    this.listLoading = false;
                //NProgress.done();
                this.$message({
                    message: 'Deleted',
                    type: 'success'
                });
                this.getOrdersProducts();
            });
            }).catch(() => {

                });
            },
            //
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                console.log(row);
                this.editForm = Object.assign({}, row);
            },
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('Update？', 'Edit', {}).then(() => {
                            this.editLoading = true;
                        //NProgress.start();
                        let para = Object.assign({}, this.editForm);
                        editOrderProducts(para).then((res) => {
                            this.editLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: 'Updated',
                            type: 'success'
                        });
                        this.$refs['editForm'].resetFields();
                        this.editFormVisible = false;
                        this.getOrdersProducts();
                    });
                    });
                    }
                });
            },
            getOrdersProducts: function () {
                let para = {
                    page: this.page,
                    name: this.filters.name
                };
                this.listLoading = true;
                //NProgress.start();
                getOrdersProducts(para).then((res) => {
                    //this.total = res.data.total;
                    this.orderProducts = res.data.data;
                this.listLoading = false;
                //NProgress.done();
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
                let para = { ids: ids };
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
            this.getOrdersProducts();
        }
    }

</script>

<style scoped>
    .card-details{
        margin-bottom:20px;
    }
</style>