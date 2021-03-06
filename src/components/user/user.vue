<template>
    <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card class="box-card">
        <!-- 搜索与添加区 -->
        <el-row :gutter="20">
            <el-col :span="9">
                <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserlist">
                <el-button slot="append" icon="el-icon-search" @click="getUserlist"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
            </el-col>
        </el-row>
        <el-table :data="userlist" border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="姓名" prop="username"></el-table-column>
            <el-table-column label="邮箱" prop="email"></el-table-column>
            <el-table-column label="电话" prop="mobile"></el-table-column>
            <el-table-column label="角色" prop="role_name"></el-table-column>
            <el-table-column label="状态">
                <template v-slot="scope">
                    {{scope.row.mg_state}}
                    <el-switch v-model="scope.row.mg_state" @change="userstateChange(scope.row)"></el-switch>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="180px">
                <template v-slot="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUser(scope.row.id)"></el-button>
                    <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                        <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[1, 2, 3, 4]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
        </el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog
    title="添加用户"
    :visible.sync="dialogVisible"
    width="50%" @close="addDialogClosed">
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 更忙用户信息的对话框 -->
    <el-dialog
        title="修改用户信息"
        :visible.sync="editDialogVisible"
        width="50%" @close="editDialogClose">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
        </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data () {
        var checkEmail = (rule, value, callback) => {
            const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            if (regEmail.test(value)) {
                return callback()
            }
            callback(new Error('请输入合法的邮箱'))
        }
        var checkMobile = (rule, value, callback) => {
            const regMobile = /^1(3|4|5|6|7|8|9)\d{9}$/
            if (regMobile.test(value)) {
                return callback()
            }
            callback(new Error('请输入合法的电话'))
        }
        return {
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 2
            },
            userlist: [],
            total: 0,
            // Dialog对话框的显示与隐藏
            dialogVisible: false,
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            addFormRules: {
                username: [
                    { required: true, message: '用户名', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '密码', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                email: [
                    { required: true, message: '邮箱', trigger: 'blur' },
                    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '电话', trigger: 'blur' },
                    { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            // 更忙用户框 的显示与隐藏
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                email: [
                    { required: true, message: '邮箱', trigger: 'blur' },
                    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '电话', trigger: 'blur' },
                    { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ]
            }
        }
    },
    created () {
        this.getUserlist()
    },
    methods: {
        // 监听 pagesize 改变
        handleSizeChange (newSize) {
            this.queryInfo.pagesize = newSize
            this.getUserlist()
        },
        // 监听要去哪一页
        handleCurrentChange (newPage) {
            // console.log(newPage)
            this.queryInfo.pagenum = newPage
            this.getUserlist()
        },
        // 监听状态栏的改变
        async userstateChange (userinfo) {
            const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
            console.log(res)
            if (res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state
                return this.$message.error('更新用户状态失败')
            }
            this.$message.success('更新用户状态成功')
        },
        // 添加用户框的关闭事件
        addDialogClosed () {
            this.$refs.addFormRef.resetFields()
        },
        addUser () {
            this.$refs.addFormRef.validate(async (valid) => {
                if (!valid) return
                const { data: res } = await this.$http.post('users', this.addForm)
                if (res.meta.status !== 201) return this.$message.error('添加用户失败')
                this.$message.success('添加用户成功')
                this.dialogVisible = false
                this.getUserlist()
            })
        },
        // 更改用户信息
        async showEditDialog (id) {
            const { data: res } = await this.$http.get('users/' + id)
            if (res.meta.status !== 200) return this.$message.error('修改信息失败!')
            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClose () {
            this.$refs.editFormRef.resetFields()
        },
        editUserInfo () {
            this.$refs.editFormRef.validate(async (valid) => {
                if (!valid) return
                const { data: res } = await this.$http.put('users/' + this.editForm.id, this.editForm)
                if (res.meta.status !== 200) return this.$message.error('更改失败')
                this.$message.success('更改成功')
                this.editDialogVisible = false
                this.getUserlist()
            })
        },
        // 删除用户
        async removeUser (id) {
            const confirmRes = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch((err) => err)
            if (confirmRes !== 'confirm') return this.$message.info('已经取消删除')
            const { data: res } = await this.$http.delete('users/' + id)
            if (res.meta.status !== 200) return this.$message.error('删除用户失败')
            this.$message.success('删除用户成功')
            this.getUserlist()
        },
        async getUserlist () {
            const { data: res } = await this.$http.get('users', { params: this.queryInfo })
            console.log(res)
            if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
            this.userlist = res.data.users
            this.total = res.data.total
        }
    }
}
</script>
<style lang="less" scoped>
.el-breadcrumb {
    margin-bottom: 15px;
    font-size: 12px;
}
.el-card {
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
}
.el-table {
    margin-top: 15px;
    font-size: 12px;
}
.el-pagination {
    margin-top: 15px;
}
</style>
