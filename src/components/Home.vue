<template>
    <el-container class="main-wrap">
        <el-aside :width="isCollapse ? '64px' : '200px'" class="main-left">
            <div class="logo-wrap">
              <div class="logo-box">
                  <img src="../assets/img/logo.png">
              </div>
            </div>
            <el-menu
                background-color="#364258"
                text-color="#fff"
                active-text-color="#409eff"
                unique-opened
                :collapse="isCollapse"
                :collapse-transition="false"
                :router="true"
                :default-active="activePath">
                <!-- 一级菜单 -->
                <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
                    <template slot="title">
                        <i :class="iconsObj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <!-- 二級菜单 -->
                    <el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children" :key="subitem.id" @click="saveNavState('/' + subitem.path)">
                        <template slot="title">
                            <i class="el-icon-menu"></i>
                            <span>{{subitem.authName}}</span>
                        </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
            <div class="toggle-button" @click="toggleCollpase">
                <i :class="isCollapse ? 'iconfont icon-muluzhankai' : 'iconfont icon-mulushouqi'"></i>
            </div>
        </el-aside>
        <el-container>
            <el-header class="topbar">
                <el-button type="info" @click="logout">退出</el-button>
            </el-header>
            <el-main class="main-right">
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
    data() {
        return {
            menuList: [],
            iconsObj: {
                '125': 'iconfont icon-yonghuguanli',
                '103': 'iconfont icon-guazaideyunzhuji',
                '101': 'iconfont icon-yunpan',
                '102': 'iconfont icon-xuniyun1',
                '145': 'iconfont icon-leixing'
            },
            isCollapse: false,
            activePath: ''
        }
    },
    created() {
        this.getMenuList();
        this.activePath = window.sessionStorage.getItem('activePath');
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push("/login");
        },
        //获取菜单
        getMenuList() {
            const $this = this;
            this.$http.get('menus').then(function(res){
                if(res.data.meta.status !== 200) return $this.$message.error(res.data.meta.msg);
                $this.menuList = res.data.data;
            })
        },
        toggleCollpase() {
            this.isCollapse = !this.isCollapse;
        },
        saveNavState(activePath) {
            window.sessionStorage.setItem('activePath', activePath);
            this.activePath = activePath;
        }
    }
}
</script>

<style lang="less" scoped>
.main-wrap{
    height: 100%;
}
.main-left{
    position: relative;
    background: #364258;
    .logo-wrap{
        height: 60px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    i{
        color: #fff;
    }
}
.topbar{
    display: flex;
    align-items: center;
    justify-content: flex-end;
    background: #f8f9fa;
}
.main-right{
    background: #fff;
}
.el-menu{
    border: none;
}
.toggle-button{
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    background-color: #2a374d;
    height: 43px;
    line-height: 40px;
    text-align: center;
    cursor: pointer;
}
</style>