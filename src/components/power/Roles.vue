<template>
  <div>
      <!-- 面包屑导航区域 -->
      <el-breadcrumb>
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary">添加角色</el-button>
            </el-col>
        </el-row>

        <!-- 角色列表区域 -->
        <el-table :data="rolesList">
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bbottom', i1 === 0 ? 'btop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                        <!-- 一级权限 -->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 二级 三级权限 -->
                        <el-col :span="19">
                            <el-row :class="[i2 === 0 ? '' : 'btop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <!-- 二级权限 -->
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                                        {{item3.authName}}
                                    </el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column label="#" type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope" width="300px">
                    <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightsDialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
        <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRightsDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotRight">确 定</el-button>
        </span>
</el-dialog>
  </div>
</template>

<script>
export default {
    data() {
        return {
            rolesList: [],
            setRightsDialogVisible: false,
            rightsList: [],
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            defKeys: [],
            roleId: ''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const {data: res} = await this.$http.get('roles')
            if(res.meta.status !== 200) return this.$message.error('获取角色列表数据失败')
            this.rolesList = res.data

           
        },
        async removeRightById(role, rightId) {
            const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err =>err)
            if(confirmResult !== 'confirm') {
                return this.$message.info('取消了删除')
            }
            const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if(res.meta.status !== 200) {
                return this.$message.error('删除权限失败')
            }
            role.children = res.data
        },
        //展示分配权限的对话框
        async showSetRightDialog(role) {
            this.roleId = role.id
            //获取所有权限的数据
            const {data: res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败')
            }
            //把获取到的权限数据保存到 data中
            this.rightsList = res.data

            //递归获取三级节点的 ID
            this.getLeafKeys(role, this.defKeys)
            this.setRightsDialogVisible = true
        },
        //通过递归的形式,获取角色下所有的三级权限的 id ,并保存到 defKeys 数组中
        getLeafKeys(node, arr) {
            if(!node.children) {
                return arr.push(node.id)
            }

            node.children.forEach(item => this.getLeafKeys(item, arr))
        },
        //监听分配权限对话框的关闭事件
        setRightDialogClosed() {
            this.defKeys = []
        },
        //点击为角色分配权限
        async allotRight() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            
            const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})

            if(res.meta.status !== 200) {
                return this.$message.error('分配权限失败')
            }

            this.$message.success('分配权限成功')
            this.getRolesList()
            this.setRightsDialogVisible = false

        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}
.btop {
    border-top: 1px solid #eee;
}
.bbottom {
    border-bottom: 1px solid #eee;
}
.vcenter {
    display: flex;
    align-items: center;
}
</style>