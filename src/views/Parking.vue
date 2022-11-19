<template>
  <div>
    <div style="margin: 10px 0">
      <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="number"></el-input>
      <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
      <el-button type="warning" @click="reset">重置</el-button>
    </div>
    <div style="margin: 10px 0">
      <el-button type="primary" @click="handleAdd">新增<i class="el-icon-circle-plus-outline"style="margin-left: 3px"></i></el-button>
      <el-popconfirm
              class="ml-5"
              confirm-button-text='确定'
              cancel-button-text='我再想想'
              icon="el-icon-info"
              icon-color="red"
              title="您确定批量删除这些数据吗？"
              @confirm="delBatch"
      >
        <el-button type="danger" slot="reference">批量删除 <i class="el-icon-remove-outline"></i></el-button>
      </el-popconfirm>

    </div>
    <el-table :data="tableData" border stripe :header-cell-class-name="'headerBg'"
              @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="ID" width="80"></el-table-column>
      <el-table-column prop="number" label="车位号码"></el-table-column>
      <el-table-column prop="phone" label="车主电话"></el-table-column>
      <el-table-column prop="owner" label="车主"></el-table-column>
      <el-table-column label="车主是否在使用">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.state" active-color="#13ce66" inactive-color="#ccc"
                     @change="changeEnable(scope.row)"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="200" align="center">
        <template slot-scope="scope">
          <el-button type="success" @click="handleEdit(scope.row)">编辑 <i class="el-icon-edit"></i></el-button>
          <el-popconfirm
                  class="ml-5"
                  confirm-button-text='确定'
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

    <el-dialog title="车位信息" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="form" label-width="80px" size="small">
        <el-form-item label="车位号码">
          <el-input v-model="form.number" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="车主电话">
          <el-input v-model="form.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="车主">
          <el-select clearable v-model="form.ownerId" placeholder="请选择车主">
            <el-option v-for="item in owners" :key="item.id" :label="item.nickname" :value="item.id"></el-option>
          </el-select>
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

  export default {
    name: "Parking",
    data() {
      return {
        form:{},
        tableData: [],
        number: '',
        phone:'',
        multipleSelection: [],
        pageNum: 1,
        pageSize: 10,
        total: 0,
        dialogFormVisible:false,
        owners:[],
        user: localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : {}
      }
    },
    created() {
      this.load()
    },
    methods: {
      load() {
        this.request.get("/parking/page", {
          params: {
            pageNum: this.pageNum,
            pageSize: this.pageSize,
            number: this.number,
          }
        }).then(res => {
          this.tableData = res.data.records
          this.total = res.data.total
        })

        this.request.get("/user/roleAll").then(res => {
          this.owners = res.data
        })
      },
      save() {
        this.request.post("/parking/", this.form).then(res => {
          if (res.code === '200') {
            this.$message.success("保存成功")
            this.dialogFormVisible = false
            this.load() //这里即使不用这个方法也能实现编辑页面的实时刷新，因为在dialog中我留下了:model="form"，实现了双向绑定
          } else {
            this.$message.error("保存失败")
          }
        })
      },
      handleAdd() {
        this.dialogFormVisible = true
        this.form = {}
      },
      handleEdit(row) {
        // this.form = Object.assign({}.row)
        this.form = JSON.parse(JSON.stringify(row))
        this.dialogFormVisible = true
      },
      changeEnable(row) {
        this.request.post("/parking/update", row).then(res => {
          if (res.code === '200') {
            this.$message.success("操作成功")
          }
        })
      },
      del(id) {
        this.request.delete("/parking/" + id).then(res => {
          if (res.code === '200') {
            this.$message.success("删除成功")
            this.load()
          } else {
            this.$message.error("删除失败")
          }
        })
      },
      handleSelectionChange(val) {
        console.log(val)
        this.multipleSelection = val
      },
      delBatch() {
        let ids = this.multipleSelection.map(v => v.id)  // [{}, {}, {}] => [1,2,3]
        this.request.post("/parking/del/batch", ids).then(res => {
          if (res.code === '200') {
            this.$message.success("批量删除成功")
            this.load()
          } else {
            this.$message.error("批量删除失败")
          }
        })
      },
      reset() {
        this.number = ""
        this.load()
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
      handleFileUploadSuccess(res) {
        console.log(res)
        this.$message.success("上传成功")
        this.load()
      },
      download(url) {
        window.open(url)
      },

    }
  }
</script>

<style scoped>

</style>
