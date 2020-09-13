<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            expand-trigger="hover"
            :options="catelist"
            :props="cateProps"
            v-model="selectedCateKeys"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="HandleTabClick"></el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 添加参数的对话框 -->
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

    <!-- 修改参数的对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 添加参数的对话框 -->
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
    data () {
        return {
            catelist: [],
            cateProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedCateKeys: [],
            activeName: [],
            manyTableData: [],
            onlyTableData: [],
            addDialogVisible: false,
            addForm: {
                attr_name: ''
            },
            addFormRules: {
                attr_name: [
                    {required: true,message: '请输入参数名称'， trigger: 'blur'}
                ]
            },
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                attr_name: [
                    {required: true, message:'请输入参数名称', trigger: 'blur'}
                ]
            }
        }
    },
    created () {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const { data:res } =await this.$http.get('categories')
            if(res.meta.status !== 200){
                return this.$message.error('获取商品分类失败')
            }
            this.catelist = res.data
            console.log(this.catelist)
        },
        handleChange() {
            this.getParamsData()
        },
        HandleTabClick() {
            console.log(this.activeName)
            this.getParamsData()
        },
        async getParamsData() {
            if (this.selectedCateKeys.leng !== 3) {
                this.selectedCateKeys = []
                return
            }
            console.log(this.selectedCateKeys)
            const { data:res } = await this.$http.get(`categories/${this.cateId}/attributes`,{
                params: { sel: this.activeName}
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数列表失败！')
            }
            console.log(res.data)
            if (this.activeName === 'many') {
                this.manyTableData = res.data
            } else{
                this.onlyTableData = res.data
            }
        },
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        addParams() {
            this.$refs.addFormRef.validate(async vaild => {
                if (!vaild) return
                const { data: res } =  await this.$http.post(`categories/${this.cateId}/attributes`,{
                    attr_name: this.addForm.attr_name,attr_sel: this.activeName
                }) 
                if(res.meta.status !== 201){
                    return this.$message.error('添加参数失败！')
                }
                this.$message.success('添加参数成功')
                this.addDialogVisible = false
                this.getParamsData()
            })
        },
        async showEditDialog(attr_id){
            const { data:res }= await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{
                params: {attr_sel: this.activeName}
            }
            )
            if (res.meta.status !==200 ) {
                  return this.$message.error('获取参数信息失败！')
            }
            this.editForm = res.data
            this.editDialogVisible = true
    },
    editDialogClosed() {
        this.$refs.editFormRef.resetFields()
    },
    async editParams() {
        this.$refs.editFormRef.validate(async valid => {
            if (!vaild) return
            const { data:res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
                attr_name: this.editForm.attr_name, attr_sel: this.activeName
            })
            if (res.meta.status !==200 ) {
                return this.$message.error('修改参数失败')
            }
            this.$message.success('修改参数成功')
            this.getParamsData()
            this.editDialogVisible = false
        })
    },
    async removeParams(attr_id) {
        const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }
        ).catch(error => error)
        if (confirmResult !== 'confirm') {
             return this.$message.info('已取消删除！')
        }  
        const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
         )

        if (res.meta.status !== 200) {
            return this.$message.error('删除参数失败！')
         }

        this.$message.success('删除参数成功！')
        this.getParamsData()  
    }
}
    computed: {
       isBtnDisabled(){
           if(this.selectedCateKeys.length !==3){
               return true
           }
           return false
       } ,
       cateId() {
           if(this.selectedCateKeys.length === 3){
               return this.selectedCateKeys[2]
           }
           return null
       }
       titleText() {
           if(this.activeName === 'many'){
               return '动态参数'
           }
           return '静态属性'
       }
    }
}
</script>

<style lang="less" scope>
.cat_opt {
  margin: 15px 0;
}
</style>