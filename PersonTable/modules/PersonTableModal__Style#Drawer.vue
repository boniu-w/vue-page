<template>
  <a-drawer
      :title="title"
      :width="800"
      placement="right"
      :closable="false"
      @close="close"
      :visible="visible"
  >

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="公司">
          <a-input placeholder="请输入公司" v-decorator="['company', validatorRules.company ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="关系ID(关联KEYWORD_TABLE的ID)">
          <a-input placeholder="请输入关系ID(关联KEYWORD_TABLE的ID)" v-decorator="['relationship', validatorRules.relationship ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="关系人姓名">
          <a-input placeholder="请输入关系人姓名" v-decorator="['relationshipPerson', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="人员类型ID(关联KEYWORD_TABLE的ID)">
          <a-input placeholder="请输入人员类型ID(关联KEYWORD_TABLE的ID)" v-decorator="['personTypeId', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注">
          <a-input placeholder="请输入备注" v-decorator="['remarks', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="时间戳">
          <a-input placeholder="请输入时间戳" v-decorator="['timeStamp', validatorRules.timeStamp ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="删除标识">
          <a-input placeholder="请输入删除标识" v-decorator="['deleteIdentifier', validatorRules.deleteIdentifier ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保留">
          <a-input placeholder="请输入保留" v-decorator="['reserve', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保留1">
          <a-input placeholder="请输入保留1" v-decorator="['reserve1', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="保留2">
          <a-input placeholder="请输入保留2" v-decorator="['reserve2', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="创建人ID">
          <a-input placeholder="请输入创建人ID" v-decorator="['introductionId', validatorRules.introductionId ]" />
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
  import moment from "moment"

  export default {
    name: "PERSON_TABLEModal",
    data () {
      return {
        title:"操作",
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
        form: this.$form.createForm(this),
        validatorRules:{
        company:{rules: [{ required: true, message: '请输入公司!' }]},
        relationship:{rules: [{ required: true, message: '请输入关系ID(关联KEYWORD_TABLE的ID)!' }]},
        timeStamp:{rules: [{ required: true, message: '请输入时间戳!' }]},
        deleteIdentifier:{rules: [{ required: true, message: '请输入删除标识!' }]},
        introductionId:{rules: [{ required: true, message: '请输入创建人ID!' }]},
        },
        url: {
          add: "/PERSON_TABLE/pERSON_TABLE/add",
          edit: "/PERSON_TABLE/pERSON_TABLE/edit",
        },
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
          this.form.setFieldsValue(pick(this.model,'company','relationship','relationshipPerson','personTypeId','remarks','timeStamp','deleteIdentifier','reserve','reserve1','reserve2','introductionId'))
		  //时间格式化
        });

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
            //时间格式化
            
            console.log(formData)
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