<template>
  <div class="app-container">
    <vue-good-table
      ref="table"
      url="/user"
      :columns="columns"
      :rows="rows"
      mode="remote"
      :totalRows="total"
      :pagination-options="{
        enabled: true
      }"
      :search-options="{
        enabled: true,
        trigger: 'enter',
        skipDiacritics: true,
        placeholder: 'Search this table',
        searchFn: searchQuery
      }"
    >
      <template slot="table-row" slot-scope="props">
        <span v-if="props.column.field == 'action'">
          <el-button
            type="success"
            size="mini"
            icon="el-icon-edit"
            @click.stop="handleUpdate(props.row)"
            >Edit</el-button
          >
          <el-button
            type="danger"
            size="mini"
            icon="el-icon-delete"
            @click.stop="handleDelete(props.row)"
            >Delete</el-button
          >
        </span>
        <span v-else>
          {{ props.formattedRow[props.column.field] }}
        </span>
      </template>
      <div slot="table-actions">
        <el-button
          type="primary"
          size="mini"
          icon="el-icon-plus"
          @click.stop="handleCreate()"
          >Add</el-button
        >
      </div>
    </vue-good-table>
    <el-dialog
      title="User Info"
      width="40%"
      :visible.sync="dialogVisible"
      center
      :close-on-click-modal="false"
    >
      <el-form
        ref="form"
        :model="userData"
        :rules="rules"
        label-width="120px"
        label-suffix=":"
        size="mini"
      >
        <el-form-item label="用户姓名" prop="name">
          <el-input
            size="small"
            v-model="userData.name"
            placeholder="请输入用户姓名"
          ></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input
            size="small"
            v-model="userData.mobile"
            placeholder="请输入手机号"
          ></el-input>
        </el-form-item>
        <el-form-item label="登录名" prop="username">
          <el-input
            :disabled="userData.id !== undefined"
            size="small"
            v-model="userData.username"
            placeholder="请输入登录名"
          ></el-input>
        </el-form-item>
        <el-form-item label="登录密码" prop="password">
          <el-input
            :disabled="userData.id !== undefined"
            type="password"
            size="small"
            v-model="userData.password"
            placeholder="请输入登录密码"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="handleSubmit">提交</el-button>
        <el-button @click="cancelForm">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getList, insertUser, updateUser, deleteUser } from "@/api/user";

export default {
  data() {
    return {
      total: 0,
      columns: [
        {
          label: "ID",
          field: "id"
        },
        {
          label: "UserName",
          field: "username",
          filterable: true
        },
        {
          label: "Name",
          field: "name"
        },
        {
          label: "Mobile",
          field: "mobile"
        },
        {
          label: "Status",
          field: "status",
          sortable: false,
          width: "120px"
        },
        {
          label: "Action",
          field: "action",
          sortable: false,
          width: "210px"
        }
      ],
      rules: {
        name: [
          { required: true, message: "用户姓名必填", trigger: "blur" },
          { max: 16, message: "用户姓名长度不能超过16个字符", trigger: "blur" }
        ],
        mobile: {
          max: 11,
          message: "手机号码不能超过11个数字",
          trigger: "change"
        },
        username: [
          { required: true, message: "登录名必填", trigger: "blur" },
          {
            min: 4,
            max: 16,
            message: "登录名应该为4-16个字符",
            trigger: "blur"
          }
        ]
      },
      listLoading: true,
      userData: {},
      dialogVisible: false,
      rows: []
    };
  },
  computed: {
    canEdit() {
      return this.userData.id === undefined;
    }
  },
  created() {
    getList().then(response => {
      this.total = response.data.total;
      this.rows = response.data.list;
    });
  },
  methods: {
    searchQuery(row, col, cellValue, searchTerm) {
      return cellValue.contains(searchTerm);
    },
    handleCreate() {
      this.userData = Object.assign({});
      this.dialogVisible = true;
    },
    handleUpdate(data) {
      console.log(data);
      this.userData = Object.assign({}, data);
      this.dialogVisible = true;
    },
    handleDelete(data) {
      this.$confirm(
        "此操作将永久删除" + data.name + "用户, 是否继续?",
        "Warning",
        {
          confirmButtonText: "Delete",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      ).then(() => {
        deleteUser(data).then(() => {
          this.$refs.table.initList();
          this.$message({
            type: "success",
            message: "用户删除成功!"
          });
        });
      });
    },
    handleSubmit() {
      this.$refs.form.validate(valid => {
        if (valid) {
          if (this.userData.id) {
            updateUser(this.userData).then(response => {
              if (response.data) {
                this.$message({
                  message: "用户修改成功",
                  type: "success"
                });
              }
              this.$refs.form.resetFields();
              this.dialogVisible = false;
              this.$refs.table.initList();
            });
          } else {
            insertUser(this.userData).then(response => {
              if (response.data) {
                this.$message({
                  message: "用户添加成功",
                  type: "success"
                });
                this.$refs.form.resetFields();
                this.dialogVisible = false;
                this.$refs.table.initList();
              }
            });
          }
        }
      });
    },
    cancelForm() {
      this.$refs.form.resetFields();
      this.dialogVisible = false;
    }
  }
};
</script>

<style></style>
