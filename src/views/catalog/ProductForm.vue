<template>
    <section>
        <el-row>
            <el-col :span="24">
                <el-card>
                    <el-tabs type="card" @tab-click="handleClick">
                        <el-tab-pane label="Product Information">
                            <el-form labelPosition="left" ref="productInformation" :rules="productFormRules"
                                     :model="productForm" label-width="120px">
                                <el-form-item label="Name" prop="name">
                                    <el-input v-model="productForm.name"></el-input>
                                </el-form-item>

                                <el-form-item label="Manufacturer" prop="manufacturer_id">
                                    <el-select v-model="productForm.manufacturer_id" clearable
                                               placeholder="Manufacturer">
                                        <el-option
                                                v-for="option in manufacturers"
                                                :key="option.id"
                                                :label="option.name"
                                                :value="option.id">
                                        </el-option>
                                    </el-select>
                                </el-form-item>
                                <el-form-item label="Status" prop="status">
                                    <el-radio-group v-model="productForm.status">
                                        <el-radio :label="1">Active</el-radio>
                                        <el-radio :label="0">Inactive</el-radio>
                                    </el-radio-group>
                                </el-form-item>
                                <el-form-item label="Description" prop="description">
                                    <vue-editor v-model="productForm.description"></vue-editor>
                                </el-form-item>
                                <el-form-item>
                                    <el-button type="primary" @click="saveProductInformation"
                                               :loading="savingProductInformation">Save
                                    </el-button>
                                </el-form-item>
                            </el-form>
                        </el-tab-pane>
                        <el-tab-pane label="Product Options">
                            <el-row :gutter="20">
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-form ref="form" :model="productOptionForm" label-width="120px"
                                             labelPosition="left">
                                        <el-form-item label="Name">
                                            <el-input v-model="productOptionForm.name"></el-input>
                                        </el-form-item>
                                        <el-form-item label="Option">
                                            <el-select v-model="productOptionForm.option_id"
                                                       placeholder="please select your zone">
                                                <el-option label="Zone one" value="shanghai"></el-option>
                                                <el-option label="Zone two" value="beijing"></el-option>
                                            </el-select>
                                        </el-form-item>
                                        <el-form-item label="price">
                                            <el-input-number placeholder="Price"></el-input-number>
                                        </el-form-item>
                                        <el-form-item label="Required">
                                            <el-switch on-text="" off-text=""
                                                       v-model="productOptionForm.status"></el-switch>
                                        </el-form-item>
                                        <el-form-item label="Description">
                                            <el-input type="textarea"
                                                      v-model="productOptionForm.description"></el-input>
                                        </el-form-item>
                                        <el-form-item>
                                            <el-button type="primary" @click="onSubmit">Add</el-button>
                                        </el-form-item>
                                    </el-form>
                                </el-col>
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-table height="250"
                                              :data="productOptions"
                                              border
                                              style="width: 100%">
                                        <el-table-column prop="option_id" label="Option" width="180">

                                        </el-table-column>
                                        <el-table-column prop="name" label="Name" width="180">

                                        </el-table-column>
                                        <el-table-column prop="description" label="Description" width="200">
                                        </el-table-column>
                                        <el-table-column prop="price" label="Price" width="180">

                                        </el-table-column>
                                    </el-table>
                                </el-col>
                            </el-row>
                        </el-tab-pane>
                        <el-tab-pane label="Images">
                            <el-row :gutter="20">
                                <el-col :sm="24" :xs="24" :md="24" :lg="12">
                                    <h3>
                                        Image</h3>
                                    <el-upload
                                            class="avatar-uploader"
                                            v-bind:action="uploadUrl"
                                            :show-file-list="false"
                                            :on-success="onProductImageUpload"
                                            :before-upload="uploadProductImage">
                                        <img v-if="productForm.image_url" :src="imageUrl" class="avatar">
                                        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                                    </el-upload>
                                </el-col>
                                <el-col :sm="24" :xs="24" :md="24" :lg="12">
                                    <h3>Other Images</h3>

                                    <el-upload
                                            :action="uploadUrl"
                                            list-type="picture-card"
                                            :fileList="productImages"
                                            :on-preview="handlePictureCardPreview"
                                            :on-remove="handleRemove">
                                        <i class="el-icon-plus"></i>
                                    </el-upload>
                                    <el-dialog v-model="productImageDialogVisible" size="tiny">
                                        <img width="100%" :src="dialogImageUrl" alt="">
                                    </el-dialog>
                                </el-col>
                            </el-row>
                        </el-tab-pane>
                        <el-tab-pane label="Attributes">
                            <el-row :gutter="20">
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-form ref="form" :model="productAttributeForm" label-width="120px"
                                             labelPosition="left">
                                        <el-form-item label="Name">
                                            <el-input v-model="productAttributeForm.value"></el-input>
                                        </el-form-item>
                                        <el-form-item label="Option">
                                            <el-select v-model="productAttributeForm.attribute_id"
                                                       placeholder="please select your zone">
                                                <el-option label="Zone one" value="shanghai"></el-option>
                                                <el-option label="Zone two" value="beijing"></el-option>
                                            </el-select>
                                        </el-form-item>
                                        <el-form-item label="price">
                                            <el-input-number placeholder="Price"></el-input-number>
                                        </el-form-item>
                                        <el-form-item label="Description">
                                            <el-input type="textarea" v-model="productAttributeForm.value"></el-input>
                                        </el-form-item>
                                        <el-form-item>
                                            <el-button type="primary" @click="onSubmit">Add</el-button>
                                        </el-form-item>
                                    </el-form>
                                </el-col>
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-table height="250"
                                              :data="productAttributes"
                                              border
                                              style="width: 100%">
                                        <el-table-column prop="option_id" label="Option" width="180">

                                        </el-table-column>
                                        <el-table-column prop="name" label="Name" width="180">

                                        </el-table-column>
                                        <el-table-column prop="description" label="Description" width="200">
                                        </el-table-column>
                                        <el-table-column prop="price" label="Price" width="180">

                                        </el-table-column>
                                    </el-table>
                                </el-col>
                            </el-row>
                        </el-tab-pane>
                        <el-tab-pane label="Discounts">
                            <el-row :gutter="20">
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-form ref="form" :model="productOptionDiscountForm" label-width="120px"
                                             labelPosition="left">
                                        <el-form-item label="Option">
                                            <el-select v-model="productOptionDiscountForm.product_option_id"
                                                       placeholder="Product Option">
                                                <el-option
                                                        v-for="option in productOptions"
                                                        :key="option.id"
                                                        :label="option.name"
                                                        :value="option.id">
                                                </el-option>
                                            </el-select>
                                        </el-form-item>
                                        <el-form-item label="value">
                                            <el-input-number placeholder="Amount"></el-input-number>
                                        </el-form-item>
                                        <el-form-item label="Validity">
                                            <el-col :span="11">
                                                <el-date-picker type="daterange" placeholder="Pick a date"
                                                                v-model="productOptionDiscountForm.validity"
                                                                style="width: 100%;"></el-date-picker>
                                            </el-col>
                                            <el-col class="line" :span="2">-</el-col>
                                        </el-form-item>
                                        <el-form-item label="Status">
                                            <el-switch on-text="Enabled" off-text="Disabled"
                                                       v-model="productOptionDiscountForm.status"></el-switch>
                                        </el-form-item>
                                        <el-form-item>
                                            <el-button type="primary" @click="onSubmit">Add</el-button>
                                        </el-form-item>
                                    </el-form>
                                </el-col>
                                <el-col :xs="24" :sm="24" :md="24" :lg="12">
                                    <el-table height="250"
                                              :data="productOptions"
                                              border
                                              style="width: 100%">
                                        <el-table-column prop="option_id" label="Option" width="180">

                                        </el-table-column>
                                        <el-table-column prop="name" label="Name" width="180">

                                        </el-table-column>
                                        <el-table-column prop="description" label="Description" width="200">
                                        </el-table-column>
                                        <el-table-column prop="price" label="Price" width="180">

                                        </el-table-column>
                                    </el-table>
                                </el-col>
                            </el-row>
                        </el-tab-pane>
                    </el-tabs>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit">Create</el-button>
                        <el-button>Cancel</el-button>
                    </el-form-item>
                </el-card>
            </el-col>
        </el-row>
    </section>
</template>

<script>
    import ElRow from "element-ui/packages/row/src/row";
    import ElCol from "element-ui/packages/col/src/col";
    import ElFormItem from "../../../node_modules/element-ui/packages/form/src/form-item";
    import {VueEditor} from 'vue2-editor'
    import {addProduct, editProduct, getManufacturers, getProduct, uploadFile} from "../../api/api";
    import ElButton from "../../../node_modules/element-ui/packages/button/src/button";
    import config from '../../config'
    import {apiBaseUrl} from "../../config/index";
    export default{
        components: {
            ElButton,
            ElFormItem,
            ElCol,
            ElRow,
            VueEditor
        },
        data() {
            return {
                savingProductInformation: false,
                updatingProductInformation: false,
                productImageDialogVisible: false,
                dialogImageUrl: '',
                formProductProcessing: false,
                formProductAttributeProcessing: false,
                formProductOptionProcessing: false,
                formProductOptionDiscountProcessing: false,
                manufacturers: [],
                productForm: {
                    id: -1,
                    name: '',
                    manufacturer_id: -1,
                    status: 0,
                    thumbnail_url: '',
                    image_url: '',
                    description: ''
                },
                productOptionForm: {
                    id: 0,
                    name: '',
                    option_id: '',
                    description: '',
                    status: 0
                },
                productOptionDiscountForm: {
                    id: 0,
                    name: '',
                    product_option_id: '',
                    price: '',
                    validity: 0
                },
                productAttributeForm: {
                    id: 0,
                    value: '',
                    attribute_id: '',
                    status: 0
                },
                productOptions: [],
                productDiscounts: [],
                productAttributes: [],
                productCategories: [],
                uploadUrl: apiBaseUrl + 'products/image/upload',
                productImages: [{
                    name: 'food.jpeg',
                    url: 'http://localhost/mine/image00.png'
                },
                    {
                        name: 'food2.jpeg',
                        url: 'http://localhost/mine/image00.png'
                    }],
                productFormRules: {
                    name: [
                        {required: true, message: 'Required', trigger: 'blur'}
                    ]
                },
            }
        },
        methods: {
            onSubmit() {
                console.log('submit!');
            },
            handleClick(){

            },
            updateProductInformation (){
                this.$refs.productInformation.validate((valid) => {
                    if (valid) {
                        this.$confirm('Update product？', 'Save', {}).then(() => {
                            this.updatingProductInformation = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.productForm);
                            editProduct(para).then((res) => {
                                this.updatingProductInformation = false;
                                //NProgress.done();
                                this.$message({
                                    message: 'Updated. ',
                                    type: 'success'
                                });
                            });
                        });
                    }
                });
            },
            saveProductInformation() {
                if (this.productForm.id !== -1) {
                    this.updateProductInformation();
                    return;
                }
                this.$refs.productInformation.validate((valid) => {
                    if (valid) {
                        this.$confirm('Add product？', 'Save', {}).then(() => {
                            this.savingProductInformation = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.productForm);
                            addProduct(para).then((res) => {
                                console.log(res);
                                this.savingProductInformation = false;
                                //NProgress.done();
                                this.$message({
                                    message: 'Saved. ',
                                    type: 'success'
                                });
                                //TODO: redirect
                                this.$router.push(`products/edit/${res.data.data.id}`);
                            });
                        });
                    }
                });
            },
            savingProductOption() {

            },
            updatingProductOption() {

            },
            savingProductAttribute() {

            },
            updatingProductAttribute() {

            },
            savingProductDiscount() {

            },
            updatingProductDiscount() {

            },
            handlePictureCardPreview () {

            },
            onProductImageUpload () {

            },
            uploadProductImage(file) {
                console.log(file);
                var data= new FormData();
                data.append('image',file);

                uploadFile({data: data,url: this.getProductImageUploadUrl(),method:'put'} ).then((res) => {
                    console.log(res);
                });
                return false;
            },
            getProductImageUploadUrl() {
                return apiBaseUrl + 'products/image/upload/' + this.productForm.id;
            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
            },
            getManufacturers(){
                getManufacturers({}).then((res) => {
                    //this.total = res.data.total;
                    this.manufacturers = res.data.data;
                    this.manufacturers.push({
                        id: 1,
                        name: 'Bata'
                    })
                    //NProgress.done();
                });
            },
            getProduct(id) {
                getProduct({id: id}).then((res) => {
                    //this.total = res.data.total;
                    this.productForm = res.data.data;
                    //NProgress.done();
                });
            }
        },
        computed: {
            getUploadUrl(){
                return apiBaseUrl + 'products/image/upload/' + this.productForm.id;
            }
        },
        mounted() {
            if (this.$route.params.id) {
                this.getProduct(this.$route.params.id)
            }
            this.getManufacturers()
        }

    }
</script>
<style scoped>
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }

    .avatar-uploader .el-upload:hover {
        border-color: #20a0ff;
    }

    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        margin: auto;
        width: 360px;
        height: 360px;
        line-height: 178px;
        text-align: center;
    }

    .avatar {
        width: 360px;
        height: 360px;
        display: block;
    }
</style>