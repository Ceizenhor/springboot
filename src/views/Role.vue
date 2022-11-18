<template>
    <div>
        <div style="margin: 10px 0">
            <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="name"></el-input>
            <!--            <el-input style="width: 200px"  placeholder="请输入邮箱"   suffix-icon="el-icon-message" class="ml-5" v-model="email"></el-input>-->
            <!--            <el-input style="width: 200px"  placeholder="请输入地址"   suffix-icon="el-icon-map-location" class="ml-5" v-model="address"></el-input>-->
            <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
            <el-button type="warning" @click="reset">重置</el-button>
        </div>
        <div style="margin: 10px 0">
            <el-button type="primary" @click="handleAdd">新增<i class="el-icon-circle-plus-outline"style="margin-left: 3px"></i></el-button>
            <el-popconfirm
                    class="ml-5"
                    confirm-button-text='好的'
                    cancel-button-text='我再想想'
                    icon="el-icon-info"
                    icon-color="red"
                    title="您确定批量删除这些数据吗？"
                    @confirm="delBatch"
            >
                <el-button type="danger" slot="reference">批量删除<i class="el-icon-remove-outline"
                                                                 style="margin-left: 3px"></i></el-button>
            </el-popconfirm>
            <!--            <el-upload-->
            <!--                    action="http://localhost:9090/user/import"-->
            <!--                    :show-file-list="false"-->
            <!--                    accept="xlsx"-->
            <!--                    :on-success="handleExcelImportSuccess"-->
            <!--                    style="display: inline-block">-->
            <!--            <el-button type="primary" class="ml-5">导入<i class="el-icon-bottom" style="margin-left: 3px" ></i></el-button>-->
            <!--            </el-upload>-->
            <!--            <el-button type="primary" @click="exp" style="margin-left: 5px">导出<i class="el-icon-top" style="margin-left: 3px" ></i></el-button>-->
        </div>
        <el-table :data="tableData" border stripe header-cell-class-name="headerBg"
                  row-key="id"
                  @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="40"></el-table-column>
            <el-table-column prop="id" label="ID" width="80"></el-table-column>
            <el-table-column prop="name" label="名称" width="500"></el-table-column>
            <el-table-column prop="flag" label="唯一标识"></el-table-column>
            <el-table-column prop="description" label="描述" width="500"></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="info" @click="selectMenu(scope.row)">分配菜单<i class="el-icon-menu"></i>
                    </el-button>
                    <el-button type="success" @click="handleEdit(scope.row)">编辑 <i class="el-icon-edit"></i></el-button>
                    <el-popconfirm
                            class="ml-5"
                            confirm-button-text='好的'
                            cancel-button-text='我再想想'
                            icon="el-icon-info"
                            icon-color="red"
                            title="您确定删除吗？"
                            @confirm="del(scope.row.id)"
                    >
                        <el-button type="danger" slot="reference">删除 <i class="el-icon-remove-outline"></i></el-button>
                    </el-popconfirm>
                </template>
            </el-table-column>
        </el-table>
        <div style="padding: 10px 0">
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="pageNum"
                    :page-sizes="[2, 5, 10, 20]"
                    :page-size="pageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </div>
        <el-dialog title="菜单分配" :visible.sync="menuDialogVisible" width="30%">
            <el-form :model="form" label-width="80px" size="small">
                <el-tree
                        :props="props"
                        :data="menuData"
                        show-checkbox
                        node-key="id"
                        ref="tree"
                        :default-expanded-keys="expends"
                        :default-checked-keys="checks">
                        <span class="custom-tree-node" slot-scope="{ node, data }">
                        <span><i :class="data.icon"/> {{ data.name }}</span>
                        </span>
                </el-tree>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="menuDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveRoleMenu">确 定</el-button>
            </div>
        </el-dialog>


        <el-dialog title="角色信息" :visible.sync="dialogFormVisible" width="30%">
            <el-form :model="form" label-width="80px" size="small">
                <el-form-item label="名称">
                    <el-input v-model="form.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="唯一标识">
                    <el-input v-model="form.flag " autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="form.description " autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="save">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    import request from "@/utils/request";

    export default {
        name: "Role",
        data() {
            return {
                tableData: [],
                total: 0,
                pageNum: 1,
                pageSize: 10,
                name: "",
                form: {},
                headerBg: 'headerBg',
                dialogFormVisible: false, //该句意思是默认不展示，那思路就是点击新增后把该值变成true
                menuDialogVisible: false,
                multipleSelection: [],
                menuData: [],
                props: {
                    label: 'name',
                },
                expends: [],
                checks: [],
                roleId: 0,
                roleFlag: '',
            }
        },
        created() {
            this.load()
        },
        methods: {
            handleAdd() {
                this.dialogFormVisible = true
                this.form = {}
            },
            handleEdit(row) {
                // this.form = Object.assign({}.row)
                this.form = JSON.parse(JSON.stringify(row))
                this.dialogFormVisible = true
            },
            del(id) {
                request.delete("/role/" + id).then(res => {
                    if (res.code === '200') {
                        this.$message.success("删除成功")
                        this.load() //这里即使不用这个方法也能实现编辑页面的实时刷新，因为在dialog中我留下了:model="form"，实现了双向绑定
                    } else {
                        this.$message.error("删除失败")
                    }
                })
            },
            handleSelectionChange(val) {
                this.multipleSelection = val
            },
            delBatch() {
                let ids = this.multipleSelection.map(v => v.id) //[{},{},{}] =>[1,2,3]
                request.post("/role/del/batch", ids).then(res => {
                    if (res.code == '200') {
                        this.$message.success("删除成功")
                        this.load() //这里即使不用这个方法也能实现编辑页面的实时刷新，因为在dialog中我留下了:model="form"，实现了双向绑定
                    } else {
                        this.$message.error("删除失败")
                    }
                })
            },
            reset() {
                this.name = ""
                this.load()
            },
            load() {
                this.request.get("/role/page", {
                    params: {
                        pageNum: this.pageNum,
                        pageSize: this.pageSize,
                        name: this.name,
                    }
                }).then(res => {
                    this.tableData = res.data.records
                    this.total = res.data.total
                })

            },
            save() {
                this.request.post("/role/", this.form).then(res => {
                    if (res.code === '200') {
                        this.$message.success("保存成功")
                        this.dialogFormVisible = false
                        this.load() //这里即使不用这个方法也能实现编辑页面的实时刷新，因为在dialog中我留下了:model="form"，实现了双向绑定
                    } else {
                        this.$message.error("保存失败")
                    }
                })
            },
            saveRoleMenu() {
                this.request.post("/role//roleMenu/" + this.roleId, this.$refs.tree.getCheckedKeys()).then(res => {
                    if (res.code === '200') {
                        this.$message.success("绑定成功")
                        this.menuDialogVisible = false
                        //操作管理员角色的时候需要重新登录
                        if (this.roleFlag === 'ROLE_ADMIN') {
                            this.$store.commit("logout")
                        }
                    } else {
                        this.$message.error(res.msg)
                    }

                })
            },
            handleSizeChange(pageSize) {
                console.log(pageSize)
                this.pageSize = pageSize
                this.load()
            },
            handleCurrentChange(pageNum) {
                console.log(pageNum)
                this.pageNum = pageNum
                this.load()
            },
            selectMenu(role) {
                this.menuDialogVisible = true
                this.roleId = role.id
                this.roleFlag = role.flag
                //请求菜单数据
                this.request.get("/menu", {}).then(res => {
                    this.menuData = res.data
                    this.expends = this.menuData.map(v => v.id)
                })
                this.request.get("/role/roleMenu/" + this.roleId).then(res => {
                    this.checks = res.data

                    this.request.get("/menu/ids").then(r => {
                        const ids = r.data
                        ids.forEach(id => {
                            if (!this.checks.includes(id)) {
                                this.$refs.tree.setChecked(id, false)
                            }
                        })
                    })
                    this.menuDialogVisible = true
                })
            },
            handleCheckChange(data, checked, indeterminate) {
                console.log(data, checked, indeterminate);
            },
        }
    }
</script>

<style>
</style>