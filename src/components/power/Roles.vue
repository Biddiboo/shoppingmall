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
              <el-button type="danger" @click="addDialogVisible=true">添加角色</el-button>
            </el-col>
          </el-row>
            <el-table :data="roleList" border stripe>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-row :class="['bdbottom' ,i1===0 ? 'bdtop' :'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                    <el-col :span="5">
                      <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                    </el-col>
                    <el-col :span="19">
                      <el-row :class="[i2===0 ? '' :'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                        <el-col :span="6">
                          <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                        </el-col>
                        <el-col :span="18">
                          <el-tag v-for="(item3) in item2.children" :key="item3.id" type="danger" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                        </el-col>
                      </el-row>
                    </el-col>
                  </el-row>
                  <pre>
                    {{scope.row}}
                  </pre>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column label="角色名称" prop="roleName"></el-table-column>
              <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
              <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                  <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
              </el-table-column>
            </el-table>
        </el-card>
        <!-- 添加用户弹窗 -->
        <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="40%" @close="addDialogClosed">
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
          <el-form-item label="角色名称" prop="roleName" label-width="100px">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc" label-width="100px">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 编辑信息弹窗 -->
        <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="40%" @close="editDialogClosed">
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="角色名称" prop="roleName" label-width="100px">
              <el-input v-model="editForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc" label-width="100px">
              <el-input v-model="editForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 分配权限弹窗 -->
        <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%" @close="setRightDialogClosed">
          <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef">
          </el-tree>
          <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      roleList: [],
      addDialogVisible: false,
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: {
        roleName: '',
        roleDesc: ''
      },
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      setRightDialogVisible: false,
      rightslist: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.roleList = res.data
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('编辑用户信息失败！')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('编辑用户信息成功')
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户，是否继续',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户信息失败！')
      }
      this.getRolesList()
      this.$message.success('删除用户信息成功')
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户，是否继续',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      role.children = res.data
      this.$message.success('删除权限成功')
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败！')
      }
      this.rightslist = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop{
  border-top: 1px solid #eee;
}

.bdbottom{
  border-bottom: 1px solid #eee;
}

.vcenter{
  display: flex;
  align-items: center;
}
</style>
