<template>
  <a-drawer
    :title="title"
    :maskClosable="true"
    :width="drawerWidth"
    placement="right"
    :closable="true"
    @close="handleCancel"
    :visible="visible"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">

    <template slot="title">
      <div style="width: 100%;">
        <span>{{ title }}</span>
        <span style="display:inline-block;width:calc(100% - 51px);padding-right:10px;text-align: right">
          <a-button @click="toggleScreen" icon="appstore" style="height:20px;width:20px;border:0px"></a-button>
        </span>
      </div>

    </template>

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="用户账号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入用户账号" v-decorator="[ 'username', validatorRules.username]" :readOnly="!!model.id"/>
        </a-form-item>

        <template >
          <a-form-item label="登陆密码" :labelCol="labelCol" :wrapperCol="wrapperCol" >
            <a-input type="password" placeholder="请输入登陆密码" v-decorator="[ 'password', validatorRules.password]" />
          </a-form-item>

          <a-form-item label="确认密码" :labelCol="labelCol" :wrapperCol="wrapperCol" v-if="!model.id" >
            <a-input type="password" @blur="handleConfirmBlur" placeholder="请重新输入登陆密码" v-decorator="[ 'confirmpassword', validatorRules.confirmpassword]"/>
          </a-form-item>
        </template>



        <a-form-item label="渠道名称" :labelCol="labelCol" :wrapperCol="wrapperCol" >
          <a-input placeholder="请输入渠道名称" v-decorator="[ 'realname', validatorRules.realname]" />
        </a-form-item>

        <a-form-item label="手机号码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入手机号码" v-decorator="[ 'phone', validatorRules.phone]" />
        </a-form-item>

        <a-form-item label="QQ" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入QQ" v-decorator="[ 'qq', {}]" />
        </a-form-item>

        <a-form-item label="微信号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入微信号" v-decorator="[ 'weixin', {}]" />
        </a-form-item>

        <a-form-item label="扣量规则" :labelCol="labelCol" :wrapperCol="wrapperCol" v-if="userInfo().parentId=='0'">
          <a-select v-decorator="[ 'deductId', validatorRules.deductId]" placeholder="请选择">
            <a-select-option :value="item.id" v-for="(item,idx) in deductList" :key="idx">{{item.roleName}}</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="分成比 %" :labelCol="labelCol" :wrapperCol="wrapperCol" >
          <a-input-number placeholder="分成比" v-decorator="[ 'profitShare', validatorRules.profitShare]"  />
          <span style="color: #EE553D;margin-left: 20px;" v-if="userInfo().parentId!='0'">设置比例不能超过 {{userInfo().profitShare}} %</span>
        </a-form-item>


        <a-form-item label="结算方式" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select v-decorator="[ 'settlementType', validatorRules.settlementType]" placeholder="请选择">
            <a-select-option :value="1">支付宝</a-select-option>
            <a-select-option :value="2">银行</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="支付宝 / 银行帐号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入支付宝 / 银行帐号" v-decorator="[ 'payAccount', {}]" />
        </a-form-item>

        <a-form-item label="收款人姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="收款人姓名" v-decorator="[ 'payName', {}]" />
        </a-form-item>

        <a-form-item label="结算银行" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入结算银行" v-decorator="[ 'bankName', {}]" />
        </a-form-item>


      </a-form>
    </a-spin>
    <depart-window ref="departWindow" @ok="modalFormOk"></depart-window>

    <div class="drawer-bootom-button" v-show="!disableSubmit">
      <a-popconfirm title="确定放弃编辑？" @confirm="handleCancel" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="confirmLoading">提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
  import pick from 'lodash.pick'
  import moment from 'moment'
  import Vue from 'vue'
  // 引入搜索部门弹出框的组件
  import departWindow from './DepartWindow'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import { getAction } from '@/api/manage'
  import {addQdUser,editQdUser,queryUserRole,queryall } from '@/api/api'
  import { disabledAuthFilter } from "@/utils/authFilter"
  import {duplicateCheck } from '@/api/api'
  import {  mapGetters } from 'vuex'
  import { axios } from '@/utils/request'

  export default {
    name: "RoleModal",
    components: {
      departWindow,
    },
    data () {
      return {
        departDisabled: false, //是否是我的部门调用该页面
        roleDisabled: false, //是否是角色维护调用该页面
        modalWidth:800,
        drawerWidth:700,
        modaltoggleFlag:true,
        confirmDirty: false,
        selectedDepartKeys:[], //保存用户选择部门id
        checkedDepartKeys:[],
        checkedDepartNames:[], // 保存部门的名称 =>title
        checkedDepartNameString:"", // 保存部门的名称 =>title
        userId:"", //保存用户id
        disableSubmit:false,
        userDepartModel:{userId:'',departIdList:[]}, // 保存SysUserDepart的用户部门中间表数据需要的对象
        dateFormat:"YYYY-MM-DD",
        deductList:[],
        validatorRules:{
          username:{
            rules: [{
              required: true, message: '请输入用户账号!'
            },{
              validator: this.validateUsername,
            }]
          },
          password:{
            rules: [{
              required: true,
              pattern:/^(?=.*[a-z])(?=.*\d).{8,}$/,
              message: '密码由8位数字、字母组成!'
            }, {
              validator: this.validateToNextPassword,
            }],
          },
          confirmpassword:{
            rules: [{
              required: true, message: '请重新输入登陆密码!',
            }, {
              validator: this.compareToFirstPassword,
            }],
          },
          realname:{rules: [{ required: true, message: '请输入用户名称!' }]},
          deductId:{rules: [{ required: true, message: '请选择扣费规则!' }]},
          phone:{rules: [{validator: this.validatePhone}]},
          profitShare:{rules: [{required: true,validator: this.validatePprofitShare }]},
          settlementType:{rules: [{ required: true, message: '请选择结算方式!' }]},
          email:{
            rules: [{
              validator: this.validateEmail
            }],
          },
          roles:{}
          //  sex:{initialValue:((!this.model.sex)?"": (this.model.sex+""))}
        },
        title:"操作",
        visible: false,
        model: {},
        roleList:[],
        selectedRole:[],
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        uploadLoading:false,
        confirmLoading: false,
        headers:{},
        form:this.$form.createForm(this),
        picUrl: "",
        url: {
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
          userWithDepart: "/sys/user/userDepartList", // 引入为指定用户查看部门信息需要的url
          userId:"/sys/user/generateUserId", // 引入生成添加用户情况下的url
          syncUserByUserName:"/process/extActProcess/doSyncUserByUserName",//同步用户到工作流
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}

    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      ...mapGetters(["userInfo"]),
      isDisabledAuth(code){
        return disabledAuthFilter(code);
      },
      //窗口最大化切换
      toggleScreen(){
        if(this.modaltoggleFlag){
          this.modalWidth = window.innerWidth;
        }else{
          this.modalWidth = 800;
        }
        this.modaltoggleFlag = !this.modaltoggleFlag;
      },
      initialRoleList(){
        queryall().then((res)=>{
          if(res.success){
            this.roleList = res.result;
          }else{
            console.log(res.message);
          }
        });
      },

      refresh () {
          this.selectedDepartKeys=[];
          this.checkedDepartKeys=[];
          this.checkedDepartNames=[];
          this.checkedDepartNameString = "";
          this.userId=""
      },
      add (deductList) {
        this.picUrl = "";
        this.refresh();
        if(deductList) this.deductList = deductList;
        this.edit({settlementType:1});
      },
      edit (record,deductList) {
        if(deductList) this.deductList = deductList;
        this.resetScreenSize(); // 调用此方法,根据屏幕宽度自适应调整抽屉的宽度
        let that = this;
        that.form.resetFields();

        that.userId = record.id;
        that.visible = true;
        that.model = Object.assign({}, record);
        console.log(that.model)
        that.$nextTick(() => {
          that.form.setFieldsValue(pick(this.model,'username','profitShare','realname','phone','qq','weixin','password','payAccount','payName','bankName','settlementType',"deductId"))
        });

      },
      //
      loadCheckedDeparts(){
        let that = this;
        if(!that.userId){return}
        getAction(that.url.userWithDepart,{userId:that.userId}).then((res)=>{
          that.checkedDepartNames = [];
          if(res.success){
            for (let i = 0; i < res.result.length; i++) {
              that.checkedDepartNames.push(res.result[i].title);
              this.checkedDepartNameString = this.checkedDepartNames.join(",");
              that.checkedDepartKeys.push(res.result[i].key);
            }
            that.userDepartModel.departIdList = that.checkedDepartKeys
          }else{
            console.log(res.message);
          }
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.disableSubmit = false;
        this.selectedRole = [];
        this.userDepartModel = {userId:'',departIdList:[]};
        this.checkedDepartNames = [];
        this.checkedDepartNameString='';
        this.checkedDepartKeys = [];
        this.selectedDepartKeys = [];
      },
      moment,
      handleSubmit () {

        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let avatar = that.model.avatar;
            if(!values.birthday){
              values.birthday = '';
            }else{
              values.birthday = values.birthday.format(this.dateFormat);
            }
            let formData = Object.assign(this.model, values);
            formData.avatar = avatar;
            formData.selectedroles = this.selectedRole.length>0?this.selectedRole.join(","):'';
            formData.selecteddeparts = this.userDepartModel.departIdList.length>0?this.userDepartModel.departIdList.join(","):'';
            let u = that.userInfo();
            if(u.parentId!="0" && formData.profitShare > u.profitShare){
              that.$message.error("分成比不能大于" + u.profitShare + "%");
              that.confirmLoading = false;
              return;
            }
            // that.addDepartsToUser(that,formData); // 调用根据当前用户添加部门信息的方法
            let obj;
            if(!this.model.id){
              formData.id = this.userId;
              obj=addQdUser(formData);
            }else{
              obj=editQdUser(formData);
            }
            obj.then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.checkedDepartNames = [];
              that.userDepartModel.departIdList = {userId:'',departIdList:[]};

              that.close();
            })

          }
        })
      },
      handleCancel () {
        this.close()
      },
      validateToNextPassword  (rule, value, callback) {
        const form = this.form;
        const confirmpassword=form.getFieldValue('confirmpassword');

        if (value && confirmpassword && value !== confirmpassword) {
          callback('两次输入的密码不一样！');
        }
        if (value && this.confirmDirty) {
          form.validateFields(['confirm'], { force: true })
        }
        callback();
      },
      compareToFirstPassword  (rule, value, callback) {
        const form = this.form;
        if (value && value !== form.getFieldValue('password')) {
          callback('两次输入的密码不一样！');
        } else {
          callback()
        }
      },
      validatePprofitShare(rule, value, callback){
        if(!value || value<0){
          callback("分成比不能少于0")

        }else if(value>100){
          callback("分成比不能大于100")
        }else {

          callback()
        }
      },
      validatePhone(rule, value, callback){
        if(!value){
          callback()
        }else{
          if(new RegExp(/^1[3|4|5|7|8][0-9]\d{8}$/).test(value)){
            var params = {
              tableName: 'sys_user',
              fieldName: 'phone',
              fieldVal: value,
              dataId: this.userId
            };
            duplicateCheck(params).then((res) => {
              if (res.success) {
                callback()
              } else {
                callback("手机号已存在!")
              }
            })
          }else{
            callback("请输入正确格式的手机号码!");
          }
        }
      },
      validateEmail(rule, value, callback){
        if(!value){
          callback()
        }else{
          if(new RegExp(/^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/).test(value)){
            var params = {
              tableName: 'sys_user',
              fieldName: 'email',
              fieldVal: value,
              dataId: this.userId
            };
            duplicateCheck(params).then((res) => {
              console.log(res)
              if (res.success) {
                callback()
              } else {
                callback("邮箱已存在!")
              }
            })
          }else{
            callback("请输入正确格式的邮箱!")
          }
        }
      },
      validateUsername(rule, value, callback){
        var params = {
          tableName: 'sys_user',
          fieldName: 'username',
          fieldVal: value,
          dataId: this.userId
        };
        duplicateCheck(params).then((res) => {
          if (res.success) {
          callback()
        } else {
          callback("用户名已存在!")
        }
      })
      },
      handleConfirmBlur  (e) {
        const value = e.target.value;
        this.confirmDirty = this.confirmDirty || !!value
      },

      normFile  (e) {
        console.log('Upload event:', e);
        if (Array.isArray(e)) {
          return e
        }
        return e && e.fileList
      },
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        this.picUrl = "";
        if (info.file.status === 'uploading') {
          this.uploadLoading = true;
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if(response.success){
            this.model.avatar = response.message;
            this.picUrl = "Has no pic url yet";
          }else{
            this.$message.warning(response.message);
          }
        }
      },
      getAvatarView(){
        return this.url.imgerver +"/"+ this.model.avatar;
      },
      // 搜索用户对应的部门API
      onSearch(){
        this.$refs.departWindow.add(this.checkedDepartKeys,this.userId);
      },

      // 获取用户对应部门弹出框提交给返回的数据
      modalFormOk (formData) {
        this.checkedDepartNames = [];
        this.selectedDepartKeys = [];
        this.checkedDepartNameString = '';
        this.userId = formData.userId;
        this.userDepartModel.userId = formData.userId;
        for (let i = 0; i < formData.departIdList.length; i++) {
          this.selectedDepartKeys.push(formData.departIdList[i].key);
          this.checkedDepartNames.push(formData.departIdList[i].title);
          this.checkedDepartNameString = this.checkedDepartNames.join(",");
        }
        this.userDepartModel.departIdList = this.selectedDepartKeys;
        this.checkedDepartKeys = this.selectedDepartKeys  //更新当前的选择keys
       },
      // 根据屏幕变化,设置抽屉尺寸
      resetScreenSize(){
        let screenWidth = document.body.clientWidth;
        if(screenWidth < 500){
          this.drawerWidth = screenWidth;
        }else{
          this.drawerWidth = 700;
        }
      },
    }
  }
</script>

<style scoped>
  .avatar-uploader > .ant-upload {
    width:104px;
    height:104px;
  }
  .ant-upload-select-picture-card i {
    font-size: 49px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }

  .ant-table-tbody .ant-table-row td{
    padding-top:10px;
    padding-bottom:10px;
  }

  .drawer-bootom-button {
    position: absolute;
    bottom: -8px;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }
</style>