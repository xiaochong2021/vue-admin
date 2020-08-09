<template>
  <div id="login">
    <div class="login-wrap">
      <ul class="menu-tap">
        <li
          :class="{ current: item.current }"
          @click="toggleMenu(item)"
          v-for="(item, index) in menuTab"
          :key="index"
        >
          {{ item.text }}
        </li>
      </ul>
      <!--表单-->
      <el-form
        :model="ruleForm"
        status-icon
        :rules="rules"
        ref="ruleForm"
        class="login-form"
        size="mini"
      >
        <el-form-item prop="username" class="item-form">
          <label>邮箱</label>
          <el-input
            type="text"
            v-model="ruleForm.username"
            autocomplete="off"
          ></el-input>
        </el-form-item>

        <el-form-item prop="password" class="item-form">
          <label>密码</label>
          <el-input
            type="password"
            v-model="ruleForm.password"
            autocomplete="off"
            maxlength="20"
            minlength="6"
          ></el-input>
        </el-form-item>

        <el-form-item
          prop="passwords"
          class="item-form"
          v-show="model === 'register'"
        >
          <label>重复密码</label>
          <el-input
            type="password"
            v-model="ruleForm.passwords"
            autocomplete="off"
            maxlength="20"
            minlength="6"
          ></el-input>
        </el-form-item>

        <el-form-item prop="code" class="item-form">
          <label>验证码</label>
          <el-row :gutter="10">
            <el-col :span="15">
              <el-input
                v-model="ruleForm.code"
                maxlength="6"
                minlength="6"
              ></el-input>
            </el-col>
            <el-col :span="9">
              <el-button type="success" class="block" @click="getSms()">获取验证码</el-button>
            </el-col>
          </el-row>
        </el-form-item>

        <el-form-item>
          <el-button
            type="danger"
            @click="submitForm('ruleForm')"
            class="block login-btn"
            :disabled="loginButtonStatus"
            >{{ model === 'login' ? "登录" : "注册"}}</el-button
          >
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
import { GetSms } from "@/api/login"
import {
  stripscript,
  validateEmail,
  validatePwd,
  validateC
} from "@/utils/validate.js";
import { reactive, ref, isRef, toRefs, onMounted } from "@vue/composition-api";
export default {
  name: "login",
  setup(props, context) {
    //验证用户名为邮箱
    let validateUsername = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入用户名"));
      } else if (!validateEmail(value)) {
        callback(new Error("用户名格式有误"));
      } else {
        callback();
      }
    };

    //验证密码
    let validatePassword = (rule, value, callback) => {
      ruleForm.password = stripscript(value);
      value = ruleForm.password;
      if (value === "") {
        callback(new Error("请输入密码"));
      } else if (validatePwd(value)) {
        callback(new Error("密码为6至20位的数字加字母"));
      } else {
        callback();
      }
    };

    //验证重复密码
    let validatePasswords = (rule, value, callback) => {
      if (model.value === "login") {
        callback();
      }
      ruleForm.passwords = stripscript(value);
      value = ruleForm.passwords;
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value != this.ruleForm.password) {
        callback(new Error("重复密码不正确"));
      } else {
        callback();
      }
    };
    //验证验证码
    let checkAge = (rule, value, callback) => {
      ruleForm.code = stripscript(value);
      value = ruleForm.code;

      if (value === "") {
        return callback(new Error("请输入验证码"));
      } else if (validateC(value)) {
        return callback(new Error("验证码格式有误"));
      } else {
        callback();
      }
    };

    const menuTab = reactive([
      { text: "登录", current: true, type: "login" },
      { text: "注册", current: false, type: "register" }
    ]);
    //模块值
    const model = ref("login");

    //登录按钮禁用状态
    const loginButtonStatus = ref(true);
    //表当绑定数据
    const ruleForm = reactive({
      username: "",
      password: "",
      passwords: "",
      code: ""
    });

    //表单验证
    const rules = reactive({
      username: [{ validator: validateUsername, trigger: "blur" }],
      password: [{ validator: validatePassword, trigger: "blur" }],
      passwords: [{ validator: validatePasswords, trigger: "blur" }],
      code: [{ validator: checkAge, trigger: "blur" }]
    });

    /**
     * 声明函数
     */
    const toggleMenu = data => {
      menuTab.forEach((elem, index) => {
        elem.current = false;
      });
      //高光
      data.current = true;
      //修改模块值
      model.value = data.type;
    };
    /**
     * 获取验证码
     */
    const getSms = (() => {
        //进行提示
        if(ruleForm.username == '') {
            context.root.$message.error('邮箱不能为空');
            return false;
        }

        if(!validateEmail(ruleForm.username)) {
            context.root.$message.error('邮箱格式有误，请重新输入!!');
            return false;
        }

        //请求的接口
        let data = {
            username:ruleForm.username,
            model: "login"
        }
        GetSms(data)
    })
    /**
     * 提交订单
     */
    const submitForm = formName => {
      context.refs[formName].validate(valid => {
        if (valid) {
          alert("submit!");
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    };

    onMounted(() => {});

    return {
      submitForm,
      toggleMenu,
      rules,
      ruleForm,
      menuTab,
      model,
      loginButtonStatus,
      getSms
    };
  }
};
</script>
<style lang="scss" scoped>
#login {
  height: 100vh;
  background-color: #344a5f;
}
.login-wrap {
  width: 300px;
  margin: auto;
}
.menu-tap {
  text-align: center;
  li {
    display: inline-block;
    width: 88px;
    line-height: 36px;
    font-size: 14px;
    color: #fff;
    border-radius: 2px;
    cursor: pointer;
  }
}
.current {
  background-color: rgba(0, 0, 0, 0.1);
}
.login-form {
  margin-top: 29px;
  label {
    display: block;
    margin-bottom: 3px;
    font-size: 14px;
    color: #fff;
  }
}
.item-form {
  margin-bottom: 13px;
}
.block {
  width: 100%;
  display: block;
}
.login-btn {
  margin-top: 19px;
}
</style>
