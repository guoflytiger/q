<template>
  <div v-if="hr" >
    <el-card class="box-card" style="margin: auto;position: absolute;
  top: 0px;bottom: 0px;left: 0px;right: 0px;width: 860px;height: 610px;">
      <div slot="header" class="clearfix">
        <span>{{hr.name}}</span>
      </div>
      <div>
        <div style="display: flex;justify-content: center">
          <el-upload
              :show-file-list="false"
              :on-success="onSuccess"
              :data="hr"
              action="/hr/userface">
            <img title="点击修改用户图像" :src="hr.userFace" style="margin-top:15px;width: 100px;height: 100px;border-radius: 50px"
                alt="">
          </el-upload>
        </div>
        <div style="display: flex;justify-content: center;flex-direction: column;">
          <el-row :gutter="20">
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">账户：
                <el-tag>{{hr.account.number}}</el-tag>
              </div>
            </el-col>
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">员工工号：
                <el-tag>{{hr.empDetails.jobNum}}</el-tag>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">用户状态：
                <el-tag>{{hr.enable}}</el-tag>
              </div>
            </el-col>
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">账户状态：
                <el-tag>{{hr.account.state}}</el-tag>
              </div>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">账户余额：
                <el-tag>{{hr.account.money}}</el-tag>
              </div>
            </el-col>
            <el-col :span="12">
              <div style="margin-left: 20%;margin-top: 10px;">开户时间：
                <el-tag>{{hr.account.createTime}}</el-tag>
              </div>
            </el-col>
          </el-row>
        </div>
        <div style="display: flex;justify-content: space-around;margin-top: 20px">
          <el-button type="primary" @click="showUpdateHrInfoView">修改信息</el-button>
          <el-button type="danger" @click="showUpdatePasswdView">修改密码</el-button>
        </div>
      </div>
    </el-card>
    <el-dialog
      title="修改用户信息"
      :visible.sync="dialogVisible"
      width="50%">
      <div  style="display: flex;justify-content: center;flex-direction: column;">
        <table>
          <tr>
            <td>
              <el-tag>姓名：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.name"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>性别：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.empDetails.sex"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>身份证号：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.empDetails.idCard"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>电话号码：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.telephone"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>邮箱：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.empDetails.email"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>民族：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.empDetails.nation"></el-input>
            </td>
          </tr>
          <tr>
            <td>
              <el-tag>居住地址：</el-tag>
            </td>
            <td>
              <el-input v-model="hr2.address"></el-input>
            </td>
          </tr>
        </table>
      </div>
      <span slot="footer" class="dialog-footer">
        <div style="display: flex;justify-content: space-around;">
          <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateHrInfo">确 定</el-button>
        </div>
      </span>
    </el-dialog>
    <el-dialog
        title="修改密码"
        :visible.sync="passwdDialogVisible"
        width="30%">
      <div>
        <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="100px"
                class="demo-ruleForm">
          <el-form-item label="请输入旧密码" prop="oldpass">
            <el-input type="password" v-model="ruleForm.oldpass" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="请输入新密码" prop="pass">
            <el-input type="password" v-model="ruleForm.pass" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="新确认密码" prop="checkPass">
            <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "empAccount",
  data() {
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'));
      } else {
        if (this.ruleForm.checkPass !== '') {
          this.$refs.ruleForm.validateField('checkPass');
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'));
      } else if (value !== this.ruleForm.pass) {
        callback(new Error('两次输入密码不一致!'));
      } else {
        callback();
      }
    };
    return {
      ruleForm: {
        oldpass: '',
        pass: '',
        checkPass: ''
      },
      rules: {
        oldpass: [
          {validator: validatePass, trigger: 'blur'}
        ],
        pass: [
          {validator: validatePass, trigger: 'blur'}
        ],
        checkPass: [
          {validator: validatePass2, trigger: 'blur'}
        ]
      },
      hr: {},
      hr2: {},
      dialogVisible: false,
      passwdDialogVisible: false
    }
  },
  mounted() {
    this.initHr();
  },
  methods: {
    onSuccess() {
      this.initHr();
    },
    updateHrInfo() {
      this.putRequest("/hr/info", this.hr2).then(resp => {
        if (resp) {
          this.dialogVisible = false;
          this.initHr();
        }
      })
    },
    showUpdateHrInfoView() {
      this.dialogVisible = true;
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.ruleForm.hrid = this.hr.id;
          this.putRequest("/hr/pass", this.ruleForm).then(resp => {
            if (resp) {
              this.getRequest("/logout");
              window.sessionStorage.removeItem("user")
              this.$store.commit('initRoutes', []);
              this.$router.replace("/");
            }
          })
        } else {
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    showUpdatePasswdView() {
      this.passwdDialogVisible = true;
    },
    initHr() {
      this.getRequest('/emp/info/getAccInfo/').then(resp => {
        if (resp) {
          this.hr = resp;
          this.hr2 = Object.assign({}, this.hr);
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
