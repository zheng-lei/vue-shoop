<template>
  <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-alert
                title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                :closable="false"
                show-icon>
            </el-alert>

            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <el-cascader
                        v-model="selectedCateKeys"
                        :options="cateList"
                        :props="cateProps"
                        @change="handleChange">
                    </el-cascader>
                </el-col>
            </el-row>

            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            <template v-slot="scope">
                                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template v-slot="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
                    <el-table :data="onlyTableData" border stripe type="expand">
                        <el-table-column type="expand">
                            <template v-slot="scope">
                                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template v-slot="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>

        <el-dialog
            :title="'添加' + titleText"
            :visible.sync="addDialogVisible"
            width="50%"
            @close="addDialogClosed"
        >
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog
            :title="'修改' + titleText"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editDialogClosed"
        >
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
  </div>
</template>

<script>
export default {
    data(){
        return {
            cateList: [],
            cateProps: {
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedCateKeys: [],
            activeName: 'many',
            manyTableData: [],
            onlyTableData: [],
            addDialogVisible: false,
            addForm: {},
            addFormRules: {
                attr_name: [
                    {required: true, message: '请输入参数名称', trigger: 'blur'}
                ]
            },
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                attr_name: [
                    {required: true, message: '请输入参数名称', trigger: 'blur'}
                ]
            },
            
        }
    },
    created(){
        this.getCateList();
    },
    methods:{
        //获取所有的商品分类列表
        getCateList(){
            this.$http.get('categories').then(res => {
                if(res.data.meta.status != 200){
                    return this.$message.error("获取商品分类失败！");
                }else{
                    this.cateList = res.data.data;
                }
            })
        },
        //级联选择框选中项变化触发
        handleChange(){
            if(this.selectedCateKeys.length != 3){
                this.selectedCateKeys = [];
                this.manyTableData = [];
                this.onlyTableData = [];
            }else{
                this.$http.get(`categories/${this.cateID}/attributes`, {
                    params:{ sel: this.activeName }
                }).then(res => {
                    if(res.data.meta.status !== 200){
                        this.$message.error("获取参数列表失败！");
                    }else{
                        this.getParamsData();
                    }
                })
            }
        },
        handleTabClick(){
            this.getParamsData();
        },
        getParamsData(){
            this.$http.get(`categories/${this.cateID}/attributes`, {
                params:{ sel: this.activeName }
            }).then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("获取参数列表失败！");
                }else{
                    res.data.data.forEach(item => {
                        item.inputVisible = false;
                        item.inputValue = '';
                        item.attr_vals = item.attr_vals ? item.attr_vals.split(",") : [];
                    })
                    if(this.activeName == "many"){
                        this.manyTableData = res.data.data;
                    }else{
                        this.onlyTableData = res.data.data;
                    }
                }
            })
        },
        addDialogClosed(){
            this.$refs.addFormRef.resetFields();
        },
        addParams(){
            const $this = this;
            this.$refs.addFormRef.validate(valid => {
                if(!valid) return;
                $this.$http.post(`categories/${$this.cateID}/attributes`,{
                    attr_name: $this.addForm.attr_name,
                    attr_sel: $this.activeName
                }).then(res => {
                    if(res.data.meta.status != 201){
                        $this.$message.error("添加参数失败！");
                    }else{
                        $this.$message.success("添加参数成功！");
                        $this.addDialogVisible = false;
                        $this.getParamsData();
                    }
                })
            })
        },
        showEditDialog(attr_id){
            this.$http.get(`categories/${this.cateID}/attributes/${attr_id}`,{
                params: {attr_sel: this.activeName}
            }).then(res => {
                if(res.data.meta.status != 200){
                    this.$message.error("获取数据失败！");
                }else{
                    this.editForm = res.data.data;
                }
                
            })
            this.editDialogVisible = true;
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields();
        },
        editParams(){
            const $this = this;
            this.$refs.editFormRef.validate(valid => {
                if(!valid) return;
                $this.$http.put(`categories/${$this.cateID}/attributes/${$this.editForm.attr_id}`,{
                    attr_name: $this.editForm.attr_name,
                    attr_sel: $this.activeName
                }).then(res => {
                    if(res.data.meta.status !== 200){
                        $this.$message.error("修改失败！");
                    }else{
                        $this.$message.success("修改成功！");
                        this.getParamsData();
                        this.editDialogVisible = false;
                    }
                })
            })
        },
        handleInputConfirm(row){
            if(row.inputValue.trim().length === 0){
                row.inputValue = "";
                row.inputVisible = false;
            }else{
                row.attr_vals.push(row.inputValue.trim());
                row.inputValue = '';
                row.inputVisible = false;
                this.saveAttrVals(row);
            }
        },
        showInput(row){
            row.inputVisible = true;
            //文本框自动获得焦点
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
        },
        handleClose(i,row){
            row.attr_vals.splice(i,1);
            this.saveAttrVals(row);
        },
        saveAttrVals(row){
            this.$http.put(`categories/${this.cateID}/attributes/${row.attr_id}`,{
                attr_name: row.attr_name,
                attr_sel: row.attr_sel,
                attr_vals: row.attr_vals.join(',')
            }).then(res => {
                if(res.data.meta.status != 200){
                    this.$message.error("修改参数失败！");
                }else{
                    this.$message.success("修改参数成功！");
                }
            })
        }
    },
    computed:{
        isBtnDisabled(){
            if(this.selectedCateKeys.length !== 3){
                return true;
            }
            return false;
        },
        //当前选中的三级分类的id
        cateID(){
            if(this.selectedCateKeys.length === 3){
                return this.selectedCateKeys[2];
            }
            return null;
        },
        titleText(){
            if(this.activeName == "many"){
                return "动态参数";
            }else{
                return "静态属性";
            }
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin: 15px 10px;
}
.el-tag{
    margin: 10px;
}
.input-new-tag{
    width: 120px;
}
</style>