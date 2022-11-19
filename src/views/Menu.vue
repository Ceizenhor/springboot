<template>
    <div>
        <div style="margin: 10px 0">
            <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="name"></el-input>
            <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
            <el-button type="warning" @click="reset">重置</el-button>
        </div>
        <div style="margin: 10px 0">
            <el-button type="primary" @click="handleAdd">新增<i class="el-icon-circle-plus-outline"
                                                              style="margin-left: 3px"></i></el-button>
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
        </div>
        <el-table :data="tableData" border stripe header-cell-class-name="headerBg"
                  row-key="id"
                  @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="40"></el-table-column>
            <el-table-column prop="id" label="ID" width="80"></el-table-column>
            <el-table-column prop="name" label="名称"></el-table-column>
            <el-table-column prop="path" label="路径"></el-table-column>
            <el-table-column prop="pagePath" label="页面路径"></el-table-column>
            <el-table-column prop="icon" label="图标" class-name="fontSize18" align="center" label-class-name="fontSize12">
                <template slot-scope="scope" >
                    <i :class="scope.row.icon"></i>
                </template>
            </el-table-column>
            <el-table-column prop="description" label="描述"></el-table-column>
            <el-table-column label="操作" width="300">
                <template slot-scope="scope">
                    <el-button type="primary" @click="handleAddChildren(scope.row.id)" v-if="!scope.row.pid && !scope.row.path">新增子菜单 <i class="el-icon-plus"></i></el-button>
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
        <el-dialog title="菜单信息" :visible.sync="dialogFormVisible" width="30%">
            <el-form :model="form" label-width="80px" size="small">
                <el-form-item label="名称">
                    <el-input v-model="form.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="路径">
                    <el-input v-model="form.path" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="页面路径">
                    <el-input v-model="form.pagePath" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item  label="图标">
                    <el-select clearable v-model="form.icon" placeholder="请选择" style="width: 100%">
                        <el-option v-for="item in options" :key="item.name" :label="item.name" :value="item.value">
                            <i :class="item.value" />{{item.name}}
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="form.description" autocomplete="off"></el-input>
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
                path: "",
                // icon: "",
                description: "",
                form: {},
                dialogFormVisible: false, //该句意思是默认不展示，那思路就是点击新增后把该值变成true
                multipleSelection: [],
                headerBg: 'headerBg',
                options:[]
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
            handleAddChildren(pid) {
                this.dialogFormVisible = true
                this.form = {}
                if (pid){
                    this.form.pid = pid
                }
                //请求图标的数据
                this.request.get("/menu/icons", {
                }).then(res => {
                    this.options = res.data

                })
            },
            handleEdit(row) {
                // this.form = Object.assign({}.row)
                this.form = JSON.parse(JSON.stringify(row))
                this.dialogFormVisible = true
                //请求图标的数据
                this.request.get("/menu/icons", {
                }).then(res => {
                    this.options = res.data

                })
            },
            del(id) {
                request.delete("/menu/" + id).then(res => {
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
                request.post("/menu/del/batch", ids).then(res => {
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
                this.request.get("/menu", {
                    params: {
                        name: this.name,
                    }
                }).then(res => {
                    this.tableData = res.data
                })
            },
            save() {
                request.post("/menu/", this.form).then(res => {
                    if (res.code === '200') {
                        this.$message.success("保存成功")
                        this.dialogFormVisible = false
                        this.load() //这里即使不用这个方法也能实现编辑页面的实时刷新，因为在dialog中我留下了:model="form"，实现了双向绑定
                    } else {
                        this.$message.error("保存失败")
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
            exp() {
                window.open("http://localhost:9090/menu/out")
            },
            handleExcelImportSuccess() {
                this.$message.success("文件上传成功")
                this.load()
            }
        }
    }
</script>

<style>
    .headerBg {
        background-color: beige !important;
    }
    .fontSize18{
        font-size: 18px;
    }
    .fontSize12{
        font-size: 12px;
    }
</style>