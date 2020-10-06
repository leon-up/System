<template>
  <div>
      <!-- 面包屑导航 -->
      <el-breadcrumb>
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            </el-col>
        </el-row>

         <!-- 表格区域 -->
    <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :tree-table="false" :expand-type="false" :show-index="true" border :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
            <i class="el-icon-success" style="color: lightgreen; font-size: 22px;" v-if="scope.row.cat_deleted === false"></i>
            <i class="el-icon-error" style="color: lightgreen; font-size: 22px;" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
    </tree-table>

    <!-- 页码条 -->
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

    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
            <el-form-item label="分类名称: " prop="cat_name">
                <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类: " prop="cat_name">
                 <el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProps" @change="parentCateChange" clearable change-on-select></el-cascader>
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
    data() {
        return {
            // 查询条件
            queryInfo: {
                page: 3,
                pagenum: 1,
                pagesize: 5
            },
            // 商品分类的数据列表，默认为空
            catelist: [],
            // 总数据条数
            total: 0,
            // 为table指定列的定义
            columns: [{
                lable: '分类名称',
                prop: 'cat_name'
            },
            {
                label: '是否有效',
                type: 'template',
                template: 'isok'
            },
            {
                label: '排序',
                type: 'template',
                template: 'order'
            },
            {
                label: '操作',
                type: 'template',
                template: 'opt'
            }],
            //控制对话框显示与隐藏
            addCateDialogVisible: false,
            //添加分类的表单数据对象
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            //添加分类表单的验证规则对象
            addCateFormRules: {
                cat_name: [
                    { required: true, message: "请输入分类名称", trigger: "blur" },
                ]
            },
            //父级分类的列表
            parentCateList: [],
            // 指定级联选择器的配置对象
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            // 选中的父级分类的Id数组
            selectedKeys: []
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        //获取商品数据
        async getCateList() {
            const {data: res} = await this.$http.get('categories', {params: this.queryInfo})
            if(res.meta.status !== 200) {
                return this.$message.error('获取数据失败')
            }
            this.catelist = res.data.result
            this.total = res.data.total
        },
        //监听 pagesize 改变
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getCateList()
        },
        //监听 pagenum 改变
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getCateList()
        },
        //添加分类对话框
        showAddCateDialog() {
            //先获取父级分类列表
            this.getParentCateList()
            //在显示对话框
            this.addCateDialogVisible = true
        },
        //获取父级分类列表
        async getParentCateList() {
            const {data: res} = await this.$http.get('categories', {params:{type: 2}})
            if(res.meta.status !== 200) {
                return this.$message.error('获取父级列表失败')
                
            }
            console.log(res.data)
            this.parentCateList = res.data
        },
        // 选择项发生变化触发这个函数
        parentCateChange() {
            if(this.selectedKeys > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                this.addCateForm.cat_level = this.selectedKeys.length
                return
            }else {
                this.addCateForm.cat_pid = 0
                this.addCateForm.cat_level = 0
            }
        },
        //监听对话框的关闭事件,重置表单数据
        addCateDialogClosed() {
            this.$refs.addCateFormRef.resetFields()
            this.selectedKeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0
        },
        // 点击按钮，添加新的分类
        addCate() {
            this.$refs.addCateFormRef.validate(async val => {
                if(!val) {
                    return 
                }
                const {data: res} = await this.$http.post('categories', this.addCateForm)
                if(res.meta.status !== 201) {
                    return this.$message.error('添加分类失败')
                }
                this.$message.success('添加分类成功')
                this.getCateList()
                this.addCateDialogVisible = false
            })
        }
        
    }

}
</script>

<style lang="less" scoped>
.treeTable {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}


</style>