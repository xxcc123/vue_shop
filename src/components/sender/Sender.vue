<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>地址管理</el-breadcrumb-item>
      <el-breadcrumb-item>地址列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.keyword" clearable @clear="getSenderList">
            <el-button slot="append" icon="el-icon-search" @click="getSenderList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加地址</el-button>
        </el-col>
      </el-row>
      <!-- 地址列表区域 -->
      <el-table :data="userlist" border stripe>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="用户名" prop="name"></el-table-column>
        <el-table-column label="手机号" prop="phone"></el-table-column>
        <el-table-column label="邮编" prop="code"></el-table-column>
        <el-table-column label="省" prop="province"></el-table-column>
        <el-table-column label="市" prop="city"></el-table-column>
        <el-table-column label="区" prop="region"></el-table-column>
        <el-table-column label="详细地址" prop="address"></el-table-column>
        <el-table-column label="图片">
          <template>
            <img src="https://pic2.zhimg.com/80/v2-cb41c39371d1d8a9b75a97b9c74bcd1d_720w.jpg" width="100px">
          </template>
        </el-table-column>
        <el-table-column label="是否默认" prop="is_default">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.is_default === '1'">是</el-tag>
            <el-tag type="danger" v-else>否</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="创建时间" prop="created_at">
          <template slot-scope="scope">
            {{scope.row.created_at | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeSenderById(scope.row.id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页器 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[ 2, 5, 10, 20]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加地址的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="name">
          <el-input v-model="addForm.name"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="addForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="编码" prop="code">
          <el-input v-model="addForm.code"></el-input>
        </el-form-item>
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader v-model="addForm.address1" :options="cityData"
                       :props="cascaderProps" clearable
                       style="width: 100%">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
    </el-dialog>
    <!-- 修改用户的对话框 -->
    <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.name" disabled></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="phone">
          <el-input v-model="editForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="编码" prop="code">
          <el-input v-model="editForm.code"></el-input>
        </el-form-item>
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader v-model="editForm.address1" :options="cityData"
                       :props="cascaderProps" clearable
                       style="width: 100%">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="editForm.address"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="editSenderInfo">确 定</el-button>
          </span>
    </el-dialog>
  </div>
</template>

<script>
  import cityData from '../order/citydata.js'

  export default {
    data () {
      // 验证手机号的规则
      var checkMobile = (rule, value, cb) => {
        // 验证手机号的正则表达式
        const regMobile = /^0{0,1}(13[0-9]|15[7-9]|153|156|18[7-9])[0-9]{8}$/
        if(regMobile.test(value)) {
          // 合法的邮箱
          return cb()
        }
        cb (new Error('请输入合法的手机号'))
      }
      return {
        // 获取地址列表的参数对象
        queryInfo: {
          keyword: '',
          // 当前的页数
          pagenum: 1,
          // 当前每页显示多少条数据
          pagesize: 5
        },
        userlist: [],
        total: 0,
        // 控制添加用户对话框的显示和隐藏
        addDialogVisible: false,
        // 添加用户的表单数据
        addForm: {
          name: '',
          phone: '',
          code: '',
          address1: '',
          address2: ''
        },
        // 添加表单的验证规则对象
        addFormRules: {
          name: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ],
          address1: [
            { required: true, message: '请选择省市区县', trigger: 'blur' }
          ],
          address2: [
            { required: true, message: '请输入详细地址', trigger: 'blur' }
          ]
        },
        // 控制修改用户对话框的显示和隐藏
        editDialogVisible: false,
        // 查询到的用户信息对象
        editForm: {},
        // 修改表单的验证规则对象
        editFormRules: {
          mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }
          ]
        },
        // 控制分配角色对话框的显示与隐藏
        setRoleDialogVisible: false,
        // 需要被分配角色的用户信息
        userInfo: {},
        // 所有角色的数据列表
        rolesList: [],
        // 已选中的角色id值
        selectedRoleId: '',
        cityData: cityData,
        // 指定级联选择器的配置对象
        cascaderProps: {
          expandTrigger: 'hover'
        }
      }
    },
    created() {
      this.getSenderList()
    },
    methods: {
      async getSenderList() {
        const { data: res } = await this.$http.get('api/sender',
          { params: this.queryInfo },
          {
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            }
          })
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('获取地址列表失败！')
        this.userlist = res.data.list
        this.total = res.data.total
      },
      // 监听pageSize改变的事件
      handleSizeChange(newSize) {
        // console.log(newSize)
        this.queryInfo.pagesize = newSize
        // 页面显示的数据条数发生变化，需要重新渲染数据列表
        this.getSenderList()
      },
      // 监听页码值改变的事件
      handleCurrentChange(newPage) {
        console.log(newPage)
        this.queryInfo.pagenum = newPage
        // 页面显示的数据条数发生变化，需要重新渲染数据列表
        this.getSenderList()
      },
      // 监听添加用户对话框的关闭事件
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      // 点击确定按钮，添加新地址
      addUser() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          // 验证通过，则可以发起添加用户的网络请求
          const { data: res } = await this.$http.post('api/sender', this.addForm)
          if(res.meta.status !== 200) {
            return this.$message.error('添加地址失败！')
          }
          this.$message.success('添加地址成功！')
          // 添加用户成功后隐藏对话框
          this.addDialogVisible = false
          // 添加用户后需要重新渲染用户列表
          this.getSenderList()
        })
      },
      // 展示编辑地址的对话框
      async showEditDialog(id) {
        this.editDialogVisible = true
        console.log(id)
        const { data: res } = await this.$http.get('/api/sender/' + id,
          {
            platformId: 19,
            clientType: 'platform'
          }, {
            headers:
              {
                opSiteId: 19
              }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('查询地址信息失败！')
        }
        this.editForm = res.data
        this.editForm.address1 = (res.data.province + ',' + res.data.city + ',' + res.data.region).split(',')
        console.log(this.editForm)
      },
      // 监听修改用户对话框的关闭事件
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      // 修改地址信息并提交
      editSenderInfo() {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) return
          // 发起修改用户信息的数据请求
          const { data: res } = await this.$http.put('api/sender/' + this.editForm.id, this.editForm)
          if (res.meta.status !== 200) {
            return this.$message.error('更新地址信息失败！')
          }
          // 关闭对话框
          this.editDialogVisible = false
          // 刷新数据列表
          this.getSenderList()
          // 提示修改成功
          this.$message.success('更新地址信息成功！')
        })
      },
      // 根据id删除对应的地址信息
      async removeSenderById(id) {
        // 弹框询问用户是否删除数据
        // 返回值是一个Promise对象，可以使用async/await对象
        const confirmResult = await this.$confirm('此操作将永久删除该地址, 是否继续?', '提示', {
          // 确定按钮的文本
          confirmButtonText: '确定',
          // 取消按钮的文本
          cancelButtonText: '取消',
          // type指的是前面的小图标
          type: 'warning'
        }).catch(err => err)
        // 如果用户确认删除，则返回值为字符串confirm
        // 如果用户取消了删除，则返回值为字符串cancel
        // console.log(confirmResult)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        // 发起删除地址的请求
        const { data: res } = await this.$http.delete('api/sender/' + id)
        if(res.meta.status !== 200) {
          return this.$message.error('删除地址失败！')
        }
        this.$message.success('删除地址成功！')
        // 重新渲染列表
        this.getSenderList()
      }
    }
  }
</script>

<style lang="less" scoped>

</style>
