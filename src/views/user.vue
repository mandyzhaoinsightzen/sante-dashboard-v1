/**
 * 系统管理 用户管理
 */
<template>
  <div class="container">
    <div class="main">
      <h3>{{$t('menu.accountmanage')}}</h3>
      <!-- 搜索筛选 -->
      <el-form :inline="true" :model="formInline" class="search">
        <el-form-item >
          <el-input v-model="formInline.username" :placeholder="$t('datatable.account')"  class="wdh-search"></el-input>
        </el-form-item>
        <!-- <el-form-item label="角色:">
          <el-select v-model="formInline.role" placeholder="请选择角色名">
            <el-option
              v-for="item in roleData"
              :key="item.id"
              :label="item.name"
              :value="item.name"
            ></el-option>
          </el-select>
        </el-form-item> -->
        <el-form-item>
          <el-button
            type="primary"
            icon="el-icon-search"
            @click="search"
            class="btnstyle marginRight10"
          >{{$t("operation.search")}}</el-button>
          <el-button type="primary" icon="el-icon-plus" @click="handleEdit()" class="btnstyle">{{$t("operation.new")}}</el-button>
        </el-form-item>
      </el-form>
      <!--列表-->
      <el-table
        @selection-change="selectChange"
        :data="userData"
        highlight-current-row
        v-loading="loading"
        border
        element-loading-text="loading"
        class="userTable"
      >
        <el-table-column align="center" sortable prop="user" :label="$t('datatable.code')" width="100"></el-table-column>
        <el-table-column align="center" sortable prop="username" :label="$t('datatable.account')" width="200"></el-table-column>
        <el-table-column align="center" sortable prop="company_name" :label="$t('datatable.organization')" width="250"></el-table-column>
        <el-table-column align="center" sortable prop="company_code"  width="250" v-if="false"></el-table-column>
        <el-table-column align="center" sortable prop="role" :label="$t('datatable.roletype')" width="250"></el-table-column>
        <el-table-column align="center" sortable prop="created_on" :label="$t('datatable.createdate')" width="150"  :formatter="formatDate"></el-table-column>
        <el-table-column align="center" sortable prop="is_active" :label="$t('datatable.status')" min-width="150">
          <template slot-scope="scope">
            <div
              :style="{'color':scope.row.is_active==true? '#00C922':'#FF4646'}"
            >{{ scope.row.is_activeSt }}</div>
          </template>
        </el-table-column>
        <el-table-column :label="$t('datatable.operation')" min-width="150" align="center">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.is_active==true?nshow:fshow"
              active-color="#5AA3E6"
              inactive-color="#CECECE"
              @change="editType(scope.$index, scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页组件 -->
      <Pagination v-bind:child-msg="pageparm" @callFather="callFather" class="pagtn"></Pagination>
      <!-- 编辑界面 -->
      <el-dialog 
        :title="isAdd==true?$t('datatable.createaccount'):$t('datatable.editaccount')"
        :visible.sync="editFormVisible" 
        width="30%"
      >
        <el-form
          label-width="100px"
          ref="editForm"
          :model="editForm"
          :rules="rules"
          style="margin-left:30px;" @click="closeDialog('edit')"
        >
          <el-form-item :label="$t('datatable.account')" :prop="$t('datatable.account')">
            <el-input
              v-model="editForm.username"
              auto-complete="off"
              :placeholder="$t('datatable.account')"
              width="80%"
            ></el-input>
          </el-form-item>
          <el-form-item :label="$t('datatable.organization')" :prop="$t('datatable.organization')">
            <el-select v-model="editForm.company_code" :placeholder="$t('datatable.organization')" @change="selectChanged">
                <el-option
                  v-for="item in orgData"
                  :key="item.code"
                  :label="item.name"
                  :value="item.code" 
                ></el-option>
              </el-select>
          </el-form-item>
          <el-form-item :label="$t('datatable.roletype')" :prop="$t('datatable.roletype')">
            <el-select v-model="editForm.role" :placeholder="$t('datatable.roletype')">
              <el-option
                v-for="item in roleData"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button  @click="closeDialog()">{{$t("operation.cancel")}}</el-button>
          <el-button
            type="primary"
            :loading="loading"
            class="title btnstyle"
            @click="submitForm('editForm')"
          >{{$t("operation.save")}}</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
// 导入请求方法
import {
  staffList,
  staffAdd,
  orgList,
  roleList,
  getOrgTypeById,
  staffStatus,
  staffDelete,
} from '../api/api'
import Pagination from "../components/Pagination";
export default {
  data() {
    return {
      nshow: true, //switch开启
      fshow: false, //switch关闭
      loading: false, //是显示加载
      isAdd:true,
      editFormVisible: false, //控制编辑页面显示与隐藏
      // 编辑与添加
      editForm: {
        username: "",
        role: "",
        company_code: "",
      },
      // rules表单验证
      rules: {
        username: [
          {
            required: true,
            message: "username",
            trigger: "blur",
          }
        ],
        // roleId: [{ required: true, message: "Role Type", trigger: "blur" }],
      },
      // 请求数据参数
      formInline: {
        page: 1,
        page_size: 10,
        username: "",
        role: ""
      },
      //用户数据
      userData: [],
      orgTypeId:'',
      orgData:[],
      roleData: [],
      // 分页参数
      pageparm: {
        currentPage: 1,
        pageSize: 10,
        total: 10
      }
    };
  },
  // 注册组件
  components: {
    Pagination
  },

  /**
   * 数据发生改变
   */
  watch: {},

  /**
   * 创建完毕
   */
  created() {
    this.getdata(this.formInline);
    let orgParams={name: "",page_size: 100000000, page:1}
    this.getOrgName(orgParams);
  },

  /**
   * 里面的方法只有被调用才会执行
   */
  methods: {
    change(e) {
      this.$forceUpdate();
    },
    getOrgName(params){
      orgList(params).then(res=>{
       this.orgData =res.results;
      })
    },
    selectChanged(vId){
      let obj = {};
      obj = this.orgData.find((item)=>{
      if(item.code === vId){
        // 筛选出匹配数据
      getOrgTypeById(item.id).then(res=>{
        this.orgTypeId=res.type;
      })
      this.getRoleType(this.orgTypeId);
       }
      });
    },
    getRoleType(group) {
      roleList(group).then(res=>{
        this.roleData=[{id:res[0].id,name:res[0].name}];
        this.editForm.role=res[0].name;
      })
    },
    formatDate(row, column) {
        // 获取单元格数据
        let data = row[column.property]
        if(data == null) {
          return null
        }
        let dt = new Date(data)
        return dt.getFullYear() + '-' + (dt.getMonth() + 1) + '-' + dt.getDate() + ' ' + dt.getHours() + ':' + dt.getMinutes() + ':' + dt.getSeconds()
    },
    // 获取数据方法
   getdata(parameter) {
      this.loading = true;
      let dataList = [];
      staffList(parameter).then(res => {
            this.loading = false;
            var datas=res.results;
            datas.forEach(function(data, index) {
            debugger;  
            const arr = {
              "user":data.user,
              "username": data.username,
              "is_active":data.is_active,
              "is_activeSt": data.is_active==true?"Normal":"Disabled",
              "role": data.role,
              "company_code": data.company_code,
              "company_name": data.company_name,
              "created_on": data.created_on
            }
            dataList.push(arr);
            });
            this.userData =dataList;
            // 分页赋值
            this.pageparm.currentPage = this.formInline.page;
            this.pageparm.pageSize = this.formInline.page_size;
            this.pageparm.total = res.results.count;
      }).catch(error=>{
             this.$message({
          			message: error,
          			type: 'error'
              });
        });
    },
    // 分页插件事件
    callFather(parm) {
      this.formInline.current_page = parm.currentPage;
      this.formInline.page_size = parm.pageSize;
      this.getdata(this.formInline);
    },
    //搜索事件
    search() {
      this.formInline.page = 1;
      this.getdata(this.formInline);
    },
    // 修改type
    editType: function(index, row) {
      this.loading = true;
      let parm = {
        user:row.user,
        is_active: ""
      };
      let active = row.is_activeSt;
      if (active == "失效") {
        parm.is_active = "true";
      } else {
        parm.is_active = "false";
      }
      // 修改状态
      staffStatus(parm).then(res => {
        this.loading = false
        this.$message({
              type: 'success',
              message: 'Change Success!'
            })
        this.getdata(this.formInline)
        })
    },
    //显示编辑界面
    handleEdit: function(index, row) {
      this.editFormVisible = true;
      if (row != undefined && row != "undefined") {
        this.isAdd=false;
        this.editForm.username = row.username;
        this.editForm.company_code = row.company_code;
        this.editForm.role = row.roleId;
        
      } else {
        this.editForm.username = "";
        this.isAdd=true;
        this.editForm.company_code = "";
        this.editForm.roleId = "";
      }
    },
    // 编辑、添加提交方法
     submitForm(editData) {
      debugger;
      this.$refs[editData].validate(valid => {
        if (valid) {
         if (this.isAdd) {
          staffAdd(this.editForm).then(res => {
            // if(res.status=="fail"){
            //   if(res.code==2011){
            //      this.$message({
          	// 				message: "wrong phone number!",
          	// 				type: 'error'
            //         });
            //   }else{
            //      this.$message({
          	// 				message: "failure!",
          	// 				type: 'error'
            //         });
            //   }
            // }else{
              this.editFormVisible = false
              this.loading = false
              this.getdata(this.formInline)
                this.$message({
                  type: 'success',
                  message: 'success!'
                })
            // }
            }).catch(error=>{
               this.$message({
          					message: "failure!",
          					type: 'error'
                    });
           });
          }else{
             staffEdit(this.editForm).then(res => {
                this.editFormVisible = false
                this.loading = false
                this.getdata(this.formInline)
                this.$message({
                  type: 'success',
                  message: 'success!'
                })
            }).catch(error=>{
               this.$message({
          					message: "failure!",
          					type: 'error'
                    });
           });
          }
        }
      })
    },
    handleClick(data, checked, node) {
      if (checked) {
        this.$refs.tree.setCheckedNodes([]);
        this.$refs.tree.setCheckedNodes([data]);
        this.unitparm.deptId = data.id;
        this.unitparm.deptName = data.name;
        //交叉点击节点
      } else {
      }
    },
    // 选择复选框事件
    selectChange(val) {
      this.selectdata = val;
    },
    // 关闭编辑、增加弹出框
    closeDialog(dialog) {
      this.editFormVisible = false;
    },
    // 删除用户
    deleteUser(index, row) {
      this.$confirm("确定要删除吗?", "信息", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        // 删除
        //   userDelete(row.id)
        //     .then(res => {
        //       if (res.success) {
        //         this.$message({
        //           type: 'success',
        //           message: '用户已删除!'
        //         })
        //         this.getdata(this.formInline)
        //       }else if (res.code==2002){
        //         //跳转到登录
        //         this.$store.commit('logout', 'false')
        //         this.$router.push({ path: '/login' })
        //         this.$message({
        //             type: 'info',
        //             message: "登录超时,请重新登录"
        //                 })
        //       }else{
        //           this.$message({
        //               type: 'info',
        //               message: res.msg
        //           })
        //       }
        //     })
      });
    },
    // 重置密码
    resetpwd(index, row) {
      this.resetpsd.userId = row.userId;
      this.$confirm("确定要重置密码吗?", "信息", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        userPwd(this.resetpsd).then(res => {
          if (res.success) {
            this.$message({
              type: "success",
              message: "重置密码成功！"
            });
            this.getdata(this.formInline);
          } else if (res.code == 2002) {
            //跳转到登录
            this.$store.commit("logout", "false");
            this.$router.push({ path: "/login" });
            this.$message({
              type: "info",
              message: "登录超时,请重新登录"
            });
          } else {
            this.$message({
              type: "info",
              message: res.msg
            });
          }
        });
      });
    }
  }
};
</script>

<style scoped>
.container {
  overflow: hidden;
  margin: 0 auto;
  width: 80%;
  display: block;
  background: #fff;
}
.main{
    position: inherit;
    padding-top: 10%;
}
.search{
  text-align: left;
}
.main h3{text-align:left;margin: 20px 0px 50px 0px;}
.el-form-item .el-select{
    width: 80%;
  }
 .el-form-item .el-input{
   width: 80%;
 }
.btnstyle {
  background: #004B87;
  color: white;
}

.marginRight10 {
  margin-right: 10px;
}
.pagtn {
  text-align: right;
  padding-right: 10%;
}
.userTable {
  border-radius: 10px 10px 0px 0px;
  width: 100%;
}
</style>

 