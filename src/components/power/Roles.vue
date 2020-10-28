<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <!--添加角色按钮区域-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <template>
        <el-table
          :data="roleList"
          border
          stripe
          >
          <!--展开列-->
          <el-table-column
          type="expand"
          >
            <template slot-scope="scope">
              <el-row :class="['bdbottom',i1 ===0?'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                <!--渲染一级权限-->
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!--渲染二级三级权限-->
                <el-col :span="19">
                  <!--通过for循环渲染二级权限-->
                  <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id"  closable @close="removeRightById(scope.row,item3.id)">
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column
            type="index">
            <template slot-scope="scope">
              <span>{{scope.$index + 1}}</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="roleName"
            label="角色名称">
          </el-table-column>
          <el-table-column
            prop="roleDesc"
            label="角色描述">
          </el-table-column>
          <el-table-column
            label="操作"
            width="300">
            <template slot-scope="scope" >
              <!--编辑按钮-->
              <el-button type="primary" icon="el-con-edit" size="mini" @click="showEdit(scope.row.id)">编辑</el-button>
              <!--删除按钮-->
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
              <!--权限分配按钮-->
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
          </el-table-column>
        </el-table>
      </template>
      <!--添加用户的对话框-->
      <el-dialog
        title="添加角色"
        :visible.sync="addRoleVisible"
        width="50%"
        @close="addRoleClosed">
        <!--内容主体区域-->
        <el-form :model="addRole" :rules="addRoleRules" ref="addRoleRef" >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRole.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRole.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮区域-->
        <span slot="footer" class="dialog-footer">
    <el-button @click="addRoleVisible = false">取 消</el-button>
    <el-button type="primary" @click="addRoles">确 定</el-button>
  </span>
      </el-dialog>
      <!--修改用户的对话框-->
      <el-dialog
        title="修改角色信息"
        :visible.sync="editRoleVisible"
        width="50%"
        @close="editRoleClosed">
        <!--内容主体-->
        <el-form :model="editRole" :rules="editRoleRules" ref="editRoleRef" >
          <el-form-item label="角色名称" prop="roleName" >
            <el-input v-model="editRole.roleName" ></el-input>
          </el-form-item>
        </el-form>
        <el-form :model="editRole" :rules="editRoleRules" ref="editRoleRef" >
          <el-form-item label="角色描述" prop="roleDesc" >
            <el-input v-model="editRole.roleDesc" ></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮-->
        <span slot="footer" class="dialog-footer">
    <el-button @click="editRoleVisible=false">取 消</el-button>
    <el-button type="primary" @click="editRoleInfo">确 定</el-button>
  </span>
      </el-dialog>
      <!--分配权限的对话框-->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
      @close="setRightDialogClosed">
        <!--树形控件-->
        <el-tree :data="rightslist" ref="treeRef" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys"></el-tree>
        <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      roleList: [],
      // 控制添加角色的对话框显示与隐藏
      addRoleVisible: false,
      // 控制修改角色对话框显示与隐藏
      editRoleVisible: false,
      // 控制设置权限对话框显示与隐藏
      setRightDialogVisible: false,
      // 修改数据的列表数据
      editRole: {
        roleId: ''
      },
      // 树形控件的属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 所有权限的数据
      rightslist: [],
      // 默认选中的节点的id值数组
      defkeys: [
      ],
      // 即将分配角色权限的id
      roleId: '',
      // 编辑角色的输入规则
      editRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ]
      },
      // 添加角色的输入规则
      addRole: {
        roleName: '',
        roleDesc: ''
      },
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    // 获取所有数据
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.roleList = res.data
      console.log(res.data)
    },
    // 添加监听添加角色对话框的关闭事件
    addRoleClosed () {
      this.$refs.addRoleRef.resetFields()
    },
    // 添加监听修改角色对话框的关闭事件
    editRoleClosed () {
      this.$refs.editRoleRef.resetFields()
    },
    // 添加角色
    addRoles () {
      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRole)
        if (res.meta.status !== 201) { return this.$message.error('获取角色列表失败') }
        this.$message.success('添加角色成功')
        this.addRoleVisible = false
        this.getRoleList()
      })
    },
    // 根据id查找数据
    async showEdit (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.editRoleVisible = true
      this.editRole = res.data
    },
    // 修改角色数据
    editRoleInfo () {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editRole.roleId, { roleName: this.editRole.roleName, roleDesc: this.editRole.roleDesc })
        console.log(res.meta.status)
        console.log(this.editRole)
        if (res.meta.status !== 200) { return this.$message.error('更新角色信息失败') }
        this.editRoleVisible = false
        this.getRoleList()
        this.$message.success('更新信息成功')
      })
    },
    // 根据id删除对应的角色信息
    async removeRoleById (id) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }
      ).catch(err => err)
      // 如果用户确定删除，则返回值为字符串 confirm
      // 如果用户取消删除，则返回值为字符串cancel
      if (confirmResult !== 'confirm') return this.$message.info('取消了删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) { return this.message.error('删除角色失败') }
      this.$message.success('删除角色成功')
      this.getRoleList()
    },
    // 根据id删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }
      ).catch(err => err)
      // 如果用户确定删除，则返回值为字符串 confirm
      // 如果用户取消删除，则返回值为字符串cancel
      if (confirmResult !== 'confirm') return this.$message.info('取消了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) { return this.message.error('删除权限失败') }
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) { return this.message.error('获取权限数据失败') }
      this.rightslist = res.data
      // 递归获取三级节点的ID
      this.getLeafkeys(role, this.defkeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id保存到defkeys数组中
    getLeafkeys (node, arr) {
      // 如果当成node节点不包含chilren属性，是三级节点
      if (!node.children) { return arr.push(node.id) }
      node.children.forEach(item =>
        this.getLeafkeys(item, arr))
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
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) { return this.message.error('分配权限失败') }
      this.$message.success('分配权限成功')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
  .el-tag{
    margin: 7px;
  }
  .bdtop{
    border-top: 1px solid #eeeeee;
  }
  .bdbottom{
    border-bottom: 1px solid #eeeeee;
  }
  .vcenter{
    display: flex;
    align-items: center;
  }
</style>
