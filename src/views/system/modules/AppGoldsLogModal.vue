<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="用户id" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'userId', validatorRules.userId]" placeholder="请输入用户id"></a-input>
        </a-form-item>
        <a-form-item label="章节id" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'chapterId', validatorRules.chapterId]" placeholder="请输入章节id"></a-input>
        </a-form-item>
        <a-form-item label="支付书币" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'golds', validatorRules.golds]" placeholder="请输入支付书币" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="用户帐号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'userAccount', validatorRules.userAccount]" placeholder="请输入用户帐号"></a-input>
        </a-form-item>
        <a-form-item label="章节名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'chapterName', validatorRules.chapterName]" placeholder="请输入章节名称"></a-input>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  
  export default {
    name: "AppGoldsLogModal",
    components: { 
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        validatorRules:{
        userId:{rules: [{ required: true, message: '请输入用户id!' }]},
        chapterId:{rules: [{ required: true, message: '请输入章节id!' }]},
        golds:{rules: [{ required: true, message: '请输入支付书币!' }]},
        userAccount:{rules: [{ required: true, message: '请输入用户帐号!' }]},
        chapterName:{},
        },
        url: {
          add: "/app/appGoldsLog/add",
          edit: "/app/appGoldsLog/edit",
        }
     
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'userId','chapterId','golds','userAccount','chapterName'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'userId','chapterId','golds','userAccount','chapterName'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>