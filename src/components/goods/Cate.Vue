<template>
    <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
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

        <tree-table :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false"> 
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scopr.row.cat_delede===false" style="color: lightgreen;"></i>
                <i class="el-icon-error" v-else style="color: lightgreen;"></i>
            </template>
            <template slot="order" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编译</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>
        </tree-table>

        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="querInfo.pagenum" :page-size="[3, 5, 10, 15]" :page-size="querInfo.pagesize" layout="total, sizes,prev, next, jumpet" :total="total">
        </el-pagination>
    </el-card>

    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" Width="50%"
    @close="addCateDialogClosede">
       <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <!-- options 用来指定数据源 -->
          <!-- props 用来指定配置对象 -->
          <el-cascader expand-trigger="hover" :options="parentCateList" :props="cascaderProps" v-model="selectedKeys" @change="parentCateChanged" clearable change-on-select>
          </el-cascader>
        </el-form-item>
         <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
      </el-form>
    </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
          querInfo: {
              type: 3,
              pagenum: 1,
              pagesize: 5
          }
            catelist: [],
            total: 0,
            colums: [
                {
                    label: '分类名称',
                    prop: 'cat_name'
                },{
                    label: '是否有效',
                    type: 'template',
                    template: 'isok'
                }, {
                    label: '排序',
                    type: 'template',
                    template: 'order'
                }
            ],
            addCateDialogVisible: false,
            addCateForm: {
                cat_name: ''
            },
            addCateFormRules: {
                cat_name: [
                    {required: true, message: '请输入分类名称', trigger: 'blur'}
                ]
            },
            parentCateList: [],
            casecaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedKeys: []
        }
    },
    created () {    
        this.getCateLIst()
    },
    methods: {
        async getCateList() {
            const { data : res} = await this.$http.get('categories',{
                params: this.querInfo
            })

            if(res.meta.status != 200){
                return this.$message.error('获取商品分类失败')
            }
            console.log(res.data);
            this.catelist = res.data.result
            this.total = res.data.total
        },
        handleSizeChange(newSize) {
         this.querInfo.pagesize = newSize  
         this.getCateList() 
        },
        handleCurrentChange(newPage) {
         this.querInfo.pagenum = newPage 
         this.getCateList() 
        },
        showAddCateDialog() {
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        async getParentCateList() {
            const {data : res } = await this.$http.get('categories',{
                params: { type: 2}
            })

            if(res.meta.status !== 200){
                return this.$message.error('获取父级分类数据失败')
            }

            console.log(res.data);
            this.parentCateList = res.data
        },

        parentCateChage() {
            console.log(this.selectedKeys)
            if(this.selectedKeys.length > 0){
                this.addCateForm.cat_pid = this.selectedKeys[
                    this.selectedKeys.length-1
                ]
                this.addCateForm.cat_pid=0
                this.addCateForm.cat_level = 0 
            }
        },
        addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClose(){
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level=0
        this.addCateForm.cat_pid = 0
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