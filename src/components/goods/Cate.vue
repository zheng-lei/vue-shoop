<template>
  <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>

            <tree-table :data="cateList" 
                :columns="columns" 
                :selection-type="false" 
                :expand-type="false" 
                show-index
                index-text="#"
                border
                :show-row-hover="false"
                class="treeTable">
                <template v-slot:isok="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
                    <i class="el-icon-error" v-else style="color: red"></i>
                </template>
                <template v-slot:order="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>
                <template v-slot:opt="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                </template>
            </tree-table>
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[3, 5, 10, 15]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>

        <el-dialog
            title="添加分类"
            :visible.sync="addCateDialogVisible"
            width="50%" @close="addCateDialogClosed">
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类：">
                    <el-cascader
                        v-model="selectedKeys"
                        :options="parentCateList"
                        :props="cascaderProps"
                        @change="parentCateChange"
                        clearable>
                    </el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
  </div>
</template>

<script>
export default {
    data(){
        return {
            //商品分类数据列表
            cateList: [],
            //查询条件
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            total: 0,
            //为table指定列
            columns: [{
                label: '分类名称',
                prop: 'cat_name'
            },{
                label: '是否有效',
                //表示，将当前列定义为模板列
                type: 'template',
                //表示当前列使用模板名称
                template: 'isok'
            },{
                label: '排序',
                //表示，将当前列定义为模板列
                type: 'template',
                //表示当前列使用模板名称
                template: 'order'
            },{
                label: '操作',
                //表示，将当前列定义为模板列
                type: 'template',
                //表示当前列使用模板名称
                template: 'opt'
            }],
            addCateDialogVisible: false,
            //添加分类的表单数据对象
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            addCateFormRules: {
                cat_name: [
                    {required: true, message: "请输入分类名称", trigger: 'blur'}
                ]
            },
            //父级分类的列表
            parentCateList: [],
            //指定级联选择器配置对象
            cascaderProps: {
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            //选中的父级分类的id数组
            selectedKeys: []
        }
    },
    created(){
        this.getCateList()
    },
    methods:{
        //获取商品分类数据
        getCateList(){
            this.$http.get('categories', {params: this.queryInfo}).then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("获取商品分类失败！");
                }else{
                    this.cateList = res.data.data.result;
                    this.total = res.data.data.total;
                }
            })
        },
        //监听pagesize
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize;
            this.getCateList();
        },
        //监听pagenum
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getCateList();
        },
        //点击按钮展示添加分类的弹框
        showAddCateDialog(){
            this.getParentCateList();
            this.addCateDialogVisible = true;
        },
        //获取父级分类的数据列表
        getParentCateList(){
            this.$http.get('categories', {params: {type: 2}}).then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("获取数据失败！");
                }else{
                    this.parentCateList = res.data.data;
                }
            })
        },
        //选择项发生变化触发
        parentCateChange(){
            if(this.selectedKeys.length > 0){
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1];
                this.addCateForm.cat_level = this.selectedKeys.length;
            }else{
                this.addCateForm.cat_pid = 0;
                this.addCateForm.cat_level = 0;

            }
        },
        //点击按钮，添加新的分类
        addCate(){
            const $this = this;
            this.$refs.addCateFormRef.validate(valid => {
                if(!valid) return;
                $this.$http.post('categories', $this.addCateForm).then(res => {
                    if(res.data.meta.status != 201){
                        $this.$message.error("添加分类失败！");
                    }else{
                        $this.$message.success("添加分类成功！");
                        $this.getCateList();
                        $this.addCateDialogVisible = false;
                    }
                })
            })
        },
        //监听对话框的关闭事件
        addCateDialogClosed(){
            this.$refs.addCateFormRef.resetFields();
            this.selectedKeys = [];
            this.addCateForm.cat_level = 0;
            this.addCateForm.cat_pid = 0;
        }
    }
}
</script>

<style lang="less" scoped>
    .treeTable{
        margin-top: 15px;
    }
    .el-cascader{
        width: 100%;
    }
</style>