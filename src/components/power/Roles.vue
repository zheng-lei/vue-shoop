<template>
  <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>

            <el-table :data="roleList" border stripe>
                <el-table-column type="expand">
                    <template v-slot="scope">
                        <el-row v-for="(item1, i1) in scope.row.children" :key='item1.id' :class="['bobottom','vcenter', i1 === 0 ? 'botop' : '']">
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="19">
                                <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="['vcenter', i2 === 0 ? '' : 'botop']">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300">
                    <template v-slot="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>

        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClosed">
            <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
  </div>
</template>

<script>
export default {
    data(){
        return {
            //所有角色列表数据
            roleList: [],
            //控制分配权限弹窗的显隐
            setRightDialogVisible: false,
            //获取所有权限数据
            rightsList: [],
            //树形控件的属性绑定对象
            treeProps: {
                label: "authName",
                children: "children"
            },
            //默认选中的节点id
            defKeys: [],
            //当前即将分配权限的角色id
            roleId: ''
        }
    },
    created(){
        this.getRolesList();
    },
    methods: {
        //获取所有角色列表
        getRolesList(){
            this.$http.get('roles').then(res => {
                if(res.data.meta.status !==200){
                    this.$message.error("获取角色列表失败！");
                }else{
                    this.roleList = res.data.data;
                }
            })
        },
        //根据id删除对应的权限
        removeRightById(role, rightId){
            this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {    
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http.delete(`roles/${role.id}/rights/${rightId}`).then(res => {
                    if(res.data.meta.status !==200){
                        this.$message.error("删除失败！");
                    }else{
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        role.children = res.data.data;
                    }
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                }); 
            })
        },
        //展示分配权限弹框
        showSetRightDialog(role){
            this.roleId = role.id;
            //获取所有权限的数据
            this.$http.get("rights/tree").then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("获取权限数据失败！");
                }else{
                    this.rightsList = res.data.data;
                }
            })
            this.getLeafKeys(role, this.defKeys);
            this.setRightDialogVisible = true;
        },
        //获取三级权限的id
        getLeafKeys(node, arr){
            if(!node.children){
                return arr.push(node.id);
            }
            node.children.forEach(item => {
                this.getLeafKeys(item, arr);
            });
        },
        //监听分配权限弹框的关闭
        setRightDialogClosed(){
            this.defKeys = [];
        },
        //为角色分配权限
        allotRights(){
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(",");
            this.$http.post(`roles/${this.roleId}/rights`,{rids: idStr}).then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("分配权限失败！");
                }else{
                    this.$message.success("分配权限成功！");
                    this.getRolesList();
                    this.setRightDialogVisible = false;
                }
            })
        }
    }
}
</script>

<style lang="less" scoped>
    .bobottom{
        border-bottom: 1px solid #eee;
    }
    .botop{
        border-top: 1px solid #eee;
    }
    .el-tag{
        margin: 7px;
    }
    .vcenter{
        display: flex;
        align-items: center;
    }
</style>