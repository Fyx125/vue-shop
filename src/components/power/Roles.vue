<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table :data="rolesList" :border="true" :stripe="true">
        <!--展开列-->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bd-bottom', i1 === 0 ? 'bd-top' : '', 'v-center']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!--一级权限-->
              <el-col :span="5">
                <el-tag @close="removeRightById(scope.row, item1.id)" :closable="true">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--二级、三级权限-->
              <el-col :span="19">
                <el-row :class="[ i2 !== 0 ? 'bd-top' : '', 'v-center']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <!--二级权限-->
                  <el-col :span="6">
                    <el-tag @close="removeRightById(scope.row, item2.id)" :closable="true" type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!--三级权限-->
                  <el-col :span="18">
                    <el-tag @close="removeRightById(scope.row, item3.id)" :closable="true" v-for="(item3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!--索引列-->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button @click="showSetRightDialog(scope.row)" size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--分配权限-->
    <el-dialog
      @close="setRightDialogClosed"
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%">
      <el-tree
        ref="treeRef"
        :data="rightsList"
        show-checkbox
        node-key="id"
        :default-expand-all="true"
        :default-checked-keys="defKeys"
        :props="treeProps">
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
  name: 'Roles',
  data () {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中节点的id数组
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    getRolesList () {
      this.$http.get('roles').then(res => {
        // console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.rolesList = res.data.data
      })
    },
    removeRightById (role, rightId) {
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete(`roles/${role.id}/rights/${rightId}`).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          // this.getRolesList()
          // 局部数据更新
          role.children = res.data.data
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    showSetRightDialog (role) {
      this.$http.get('rights/tree').then(res => {
        console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.rightsList = res.data.data
      })
      this.getLeafKeys(role, this.defKeys)
      this.roleId = role.id
      this.setRightDialogVisible = true
    },
    // 递归形式获取所有三级权限id
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    allotRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const idStr = keys.join(',')
      this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr }).then(res => {
        console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.$message.success(res.data.meta.msg)
        this.getRolesList()
        this.setRightDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 8px;
  }
  .bd-top {
    border-top: 1px solid #eee;
  }
  .bd-bottom {
    border-bottom: 1px solid #eee;
  }
  .v-center {
    display: flex;
    align-items: center;
  }
</style>
