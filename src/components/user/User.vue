<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <el-table :data="userList" border stripe>
                <el-table-column type="index" width="50"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态" prop="mg_state">
                    <template v-slot="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180">
                    <template v-slot="scope">
                        <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.id)"></el-button>
                        <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeUserById(scope.row.id)"></el-button>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" size="mini" icon="el-icon-setting" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>

        <!-- 添加用户的弹框 -->
        <el-dialog
            title="添加用户"
            :visible.sync="addDialogVisible"
            width="50%"
            @close="addDialogClosed">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password" ></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 编辑用户的弹框 -->
        <el-dialog
            title="修改用户"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUserInfo">确 定</el-button>
            </span>
        </el-dialog>

        <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisible"
            width="50%"
            @close="setRoleDialogClosed">
            <div>
                <p>当前的用户：{{userInfo.username}}</p>
                <p>当前的角色：{{userInfo.role_name}}</p>
                <p>分配新角色：
                    <el-select v-model="selectedRoleId" placeholder="请选择">
                        <el-option
                            v-for="item in rolesList"
                            :key="item.id"
                            :label="item.roleName"
                            :value="item.id">
                        </el-option>
                    </el-select>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        var checkEmail = (rule, value, callback) => {
            const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9])+(\.[a-zA-Z0-9_-])+/;
            if(regEmail.test(value)){
                callback();
            }else{
                callback(new Error('请输入合法的邮箱！'));
            }
        }
        var checkMobile = (rule, value, callback) => {
            const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/;
            if(regMobile.test(value)){
                callback();
            }else{
                callback(new Error('请输入合法的手机！'));
            }
        }
        return {
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 1
            },
            userList: [],
            total: 0,
            //控制添加用户的弹框的显示隐藏
            addDialogVisible: false,
            addForm: {
                username: "",
                password: "",
                email: "",
                mobile: ""
            },
            addFormRules: {
                username: [
                    {required: true, message: "请输入用户名", trigger: 'blur'},
                    {min: 3, max: 10, message: "用户名长度在3~10之间", trigger: 'blur'}
                ],
                password: [
                    {required: true, message: "请输入密码", trigger: 'blur'},
                    {min: 6, max: 15, message: "密码长度在6~15之间", trigger: 'blur'}
                ],
                email: [
                    {required: true, message: "请输入邮箱", trigger: 'blur'},
                    {validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    {required: true, message: "请输入手机", trigger: 'blur'},
                    {validator: checkMobile, trigger: 'blur'}
                ]
            },
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                email: [
                    {required: true, message: "请输入邮箱", trigger: 'blur'},
                    {validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    {required: true, message: "请输入手机", trigger: 'blur'},
                    {validator: checkMobile, trigger: 'blur'}
                ]
            },
            //控制分配角色弹框的显隐
            setRoleDialogVisible: false,
            //需要被分配角色的用户信息
            userInfo: {},
            //所有角色的数据列表
            rolesList: [],
            //选中的角色的id
            selectedRoleId: ''
        }
    },
    created(){
        this.getUserList();
    },
    methods: {
        //获取用户列表
        getUserList(){
            const $this = this;
            this.$http.get('users',{params: this.queryInfo}).then(function(res){
                if(res.data.meta.status != 200) return $this.$message.error("获取用户列表失败！");
                $this.userList = res.data.data.users;
                console.log(res.data.data)
                $this.total = res.data.data.total;
            })
        },
        //监听pageSize的改变
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize;
            this.getUserList();
        },
        //监听页码值的改变
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getUserList();
        },
        //监听switch开关的状态
        userStateChanged(userinfo){
            console.log(userinfo)
            const $this =this;
            this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`).then(function(res){
                if(res.data.meta.status != 200){
                    userinfo.mg_state = !userinfo.mg_state;
                    return $this.$message.error("更新用户状态失败！");
                }
                $this.$message.success("更新用户状态成功！");
            })
        },
        //监听添加用户弹窗的关闭
        addDialogClosed(){
            this.$refs.addFormRef.resetFields();
        },
        //点击按钮添加用户
        addUser(){
            this.$refs.addFormRef.validate(valid => {
                if(!valid) return;
                this.$http.post('users',this.addForm).then((res) => {
                    if(res.data.meta.status !== 201){
                        this.$message.error("添加用户失败！");
                    }else{
                        this.$message.success("添加成功！");
                        this.addDialogVisible = false;
                        this.getUserList();
                    }
                })
            })
        },
        //展示编辑用户的弹窗
        showEditDialog(id){
            this.editDialogVisible = true;
            this.$http.get("users/" + id).then(res => {
                console.log(res)
                if(res.data.meta.status != 200){
                    return this.$message.error("查询用户信息失败！");
                }else{
                    this.editForm = res.data.data;
                }
            })
        },
        //监听修改用户弹框的关闭事件
        editDialogClosed(){
            this.$refs.editFormRef.resetFields();
        },
        //修改用户信息
        editUserInfo(){
            this.$refs.editFormRef.validate(valid => {
                if(!valid) return;
                this.$http.put("users/" + this.editForm.id, {email: this.editForm.email, mobile: this.editForm.mobile}).then(res => {
                    if(res.data.meta.status !== 200){
                        this.$message.error("更新用户信息失败！");
                    }else{
                        this.editDialogVisible = false;
                        this.getUserList();
                        this.$message.success("更新成功！");
                    }
                })
            })
        },
        //根据id删除对应的用户
        removeUserById(id){
            this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http.delete('users/' + id).then(res => {
                    if(res.data.meta.status !==200){
                        this.$message.error("删除失败！");
                    }else{
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        this.getUserList();
                    }
                })
                
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
        },
        //展示分配角色弹框
        setRole(userInfo){
            this.userInfo = userInfo;
            this.$http.get("roles").then(res => {
                if(res.data.meta.status !== 200){
                    this.$message.error("获取角色失败！");
                }else{
                    this.rolesList = res.data.data;
                }
            })
            this.setRoleDialogVisible = true;
        },
        //分配角色
        saveRoleInfo(){
            if(!this.selectedRoleId){
                this.$message.error("请选择要分配的角色！");
            }else{
                this.$http.put(`users/${this.userInfo.id}/role`, {rid: this.selectedRoleId}).then(res => {
                    if(res.data.meta.status != 200){
                        this.$message.error("更新角色失败！");
                    }else{
                        this.getUserList();
                        this.setRoleDialogVisible = false;
                        this.$message.success("更新角色成功！");
                    }
                })
            }
        },
        //监听分配角色弹框关闭事件
        setRoleDialogClosed(){
            this.selectedRoleId = '';
            this.userInfo = {};
        }
    }
}
</script>

<style lang="less" scoped>

</style>