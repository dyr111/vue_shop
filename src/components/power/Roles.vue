<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button
            type="primary"
            @click="addDialogVisible=true"
          >添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table
        :data="rolelist"
        border
        stripe
      >
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',i1 === 0 ? 'bdtop':'','vcenter']"
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row,item1.id)"
                >{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '':'bdtop','vcenter']"
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row,item3.id)"
                    >
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>

            <!-- <pre>
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="角色名称"
          prop="roleName"
        ></el-table-column>
        <el-table-column
          label="角色描述"
          prop="roleDesc"
        ></el-table-column>
        <el-table-column
          label="操作"
          width="300px"
        >
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defkeys"
        ref="treeRef"
      ></el-tree>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="allotRights"
        >确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="80px"
      >
        <el-form-item
          label="角色名称"
          prop="roleName"
        >
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item
          label="角色描述"
          prop="roleDesc"
        >
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addUser"
        >确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色的对话框 -->
    <el-dialog
      title="修改角色信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="80px"
      >
        <el-form-item label="角色ID">
          <el-input
            v-model="editForm.roleId"
            disabled
          ></el-input>
        </el-form-item>
        <el-form-item
          label="角色名称"
          prop="roleName"
        >
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item
          label="角色描述"
          prop="roleDesc"
        >
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="editUserInfo"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
data () {
  return {
    // 所有角色列表数据
    rolelist: [],
    // 控制添加角色对话框的显示与隐藏
    addDialogVisible: false,
    addForm: {
      roleName: '',
      roleDesc: ''
    },
    addFormRules: {
      roleName: [
        {
          required: true,
          message: '请输入角色名称',
          trigger: 'blur'
        }, { min: 3, max: 10, message: '角色名称的长度在3~10个字符之间' }
      ],
      roleDesc: [
        {
          required: true,
          message: '请输入角色描述',
          trigger: 'blur'
        }, { min: 6, max: 12, message: '角色名称的长度在6~12个字符之间' }
      ]
    },
    // 控制修改角色对话框的显示与隐藏
    editDialogVisible: false,
    // 查询到的角色信息对象
    editForm: {},
    editFormRules: {
      roleName: [
        { required: true, message: '请输入角色名称', tigger: 'blur' }
      ],
      roleDesc: [
        { required: true, message: '请输入角色描述', tigger: 'blur' }
      ]
    },
    // 控制分配权限对话框的显示与隐藏
    setRightDialogVisible: false,
    // 所有权限的数据
    rightslist: [],
    // 树形控件的属性绑定对象
    treeProps: {
        label: 'authName',
        children: 'children'
    },
    // 默认选中的节点ID值数组
    defkeys: [],
    // 当前即将分配权限的角色id
    roleId: ''
  }
},
created () {
  this.getRolesList()
},
methods: {
  // 获取所有角色的列表
  async getRolesList () {
    const { data: res } = await this.$http.get('roles')
    if (res.meta.status !== 200) {
      return this.$message.error('获取角色列表失败!')
    }

    this.rolelist = res.data
    // console.log(this.rolelist)
  },
  addDialogClosed () {
    this.$refs.addFormRef.resetFields()
  },
  // 点击按钮，添加新角色
  addUser () {
    this.$refs.addFormRef.validate(async valid => {
     // console.log(valid)
     if (!valid) return
     // 发起添加角色的网络请求
     const { data: res } = await this.$http.post('roles', this.addForm)
     if (res.meta.status !== 201) {
       this.$message.error('添加角色失败!')
     }
     this.$message.success('添加角色成功!')
     this.addDialogVisible = false
     this.getRolesList()
    })
  },
  // 展示编辑角色的对话框
  async showEditDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.message.error('查询用户信息失败!')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改角色信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId,
        {
        roleName: this.editForm.roleName,
        roleDesc: this.editForm.roleDesc
        })
          console.log(this.editForm.roleId)
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败!')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新角色信息成功!')
       })
    },
    // 根据ID删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否确认删除
          const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除!')
        }

        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败!')
    }
    // this.getRolesList()
    role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')

      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }

      // 把获取到的权限数据保存到data中
      this.rightslist = res.data
      console.log(this.rightslist)

      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defkeys)
      console.log(this.defkeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id并保存到defkeys数组中
    getLeafKeys (node, arr) {
      // 如果当前node节点不包括children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item =>
      this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defkeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败!')
      }
      this.$message.success('分配权限成功!')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
// .el-button {
//     margin-bottom: 20px;
// }
.el-tag{
  margin: 7px;
}
.bdtop{
 border-top:1px solid #eee;
}
.bdbottom{
 border-bottom:1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>