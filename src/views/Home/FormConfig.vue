<template lang="pug">
  .form-config
    el-form(
      ref="form"
      :rules="rules"
      :model="formData"
      label-width="110px"
      size="small"
    )
      el-form-item(label="swagger地址:" prop="url")
        div(style="display: flex;")
          el-input(v-model="formData.url" placeholder="请输入swagger地址")
          el-button(style="margin-left:20px;" type="primary" @click="onSubmit") 查看接口列表
      el-form-item(label="模板代码")
        div(class="container-textarea")
          el-input(v-model="formData.template" type="textarea" placeholder="请输入api函数上方的代码（可选）")
          CodeTemplate(:codes="templateResult")
      el-form-item(label="get类型模板" prop="getFormatter")
        div(class="container-textarea")
          ShowParams(:singleObject="getParamsObject")
          el-input(v-model="formData.getFormatter" type="textarea" placeholder="get方法请求模板")
          CodeTemplate(:codes="getFormatterResult")
          //- el-input(v-model="getFormatterResult" :disabled="true" type="textarea" placeholder="get方法请求模板")
      el-form-item(label="post类型模板" prop="postFormatter")
        div(class="container-textarea")
          ShowParams(:singleObject="postParamsObject")
          el-input(v-model="formData.postFormatter" type="textarea" placeholder="post方法请求模板")
          CodeTemplate(:codes="postFormatterResult")
      el-select(v-model="testSelect")
        el-option(label="测试" value="1")
</template>
<script>
  import CodeTemplate from './CodeTemplate'
  // import * as monaco from 'monaco-editor'
  import ShowParams from './components/ShowParams'
  export default {
    name: 'FormConfig',
    components: {
      CodeTemplate,
      ShowParams
    },
    data() {
      const rules = {
        url: [{ required: true, message: '请输入swagger地址', trigger: 'blur' }],
        getFormatter: [
          {
            required: true,
            message: '请输入get类型模板，刷新页面可重置模板内容',
            trigger: 'blur'
          }
        ],
        postFormatter: [
          {
            required: true,
            message: '请输入post类型模板，刷新页面可重置模板内容',
            trigger: 'blur'
          }
        ]
      }
      return {
        rules,
        testSelect: '',
        formData: {
          url: 'http://test-simba-ops.startdtapi.com/swagger-ui.html#',
          template: `/**
      * 以下代码属于自动生成，请勿手动修改
      */
      import request from '@/utils/request'

      // isApi的值可以为mock、api的便于整个文件的修改
      const isApi = 'api'
      `,
          getFormatter: `export async function {functionName}({pathParams.length !== 0 ? (queryParams.length !== 0 ? lB+pathParams.join(',')+', ...params'+rB : lB+pathParams+rB) : (queryParams.length !== 0 ? 'params' :  '')}) {
          return request(\`/$\{@isApi}{path}\`,{
          method: '{method}',
          {queryParams.length !== 0 ? 'data:params' :  ''}
        })
      }`,
          postFormatter: `export async function {functionName}({pathParams.length !== 0 ? (bodyParams.length !== 0 ? lB+pathParams.join(',')+', ...params'+rB : lB+pathParams+rB) : (bodyParams.length !== 0 ? 'params' :  '')}) {
          return request(\`/$\{@isApi}{path}\`,{
          method: '{method}',
          {bodyParams.length !== 0 ? 'data:params' :  ''}
        })
      }`
        },
        templateResult: '',
        getFormatterResult: '',
        postFormatterResult: '',
        reg: /{[\w.!=?:(),/'$"+\[\] ]+}/g,
        variableReg: /^\$(\w+)\s*=\s*(\S+)\s*/g,
        getParamsObject: {
          method: 'Get',
          path: '/ops/supply/getById/{supplyId}',
          description: '根据补数据id获取补数据实例',
          tableData: [
            {
              functionName: 'getSupplyGetById',
              pathParams: "['supplyId']",
              method: 'GET',
              path: '/ops/supply/getById/${supplyId}',
              queryParams: "['projectId']",
              bodyParams: '[]',
              headerParams: '[]',
              rB: '{',
              lB: '}'
            }
          ]
        },
        postParamsObject: {
          method: 'Post',
          path: '/ops/baseline/add',
          description: '新增基线',
          tableData: [
            {
              functionName: 'postBaselineAdd',
              method: 'POST',
              path: '/ops/baseline/add',
              queryParams: "['projectId']",
              pathParams: "['supplyId']",
              bodyParams: "['alertId', 'alertOpen', 'alertTime']",
              headerParams: '[]',
              rB: '{',
              lB: '}'
            }
          ]
        }
      }
    },
    watch: {
      formData: {
        deep: true,
        immediate: true,
        handler(value) {
          this.$emit('emitFormData', value)
        }
      },
      'formData.getFormatter': {
        handler: function(value) {
          const $ = {}
          const replacedStr = this.variableEval(value, $)
          const functionName = 'getSupplyGetById'
          const path = '/ops/supply/getById/${supplyId}'
          const method = 'GET'
          const pathParams = ['supplyId']
          const queryParams = ['projectId']
          const headerParams = ['']
          const bodyParams = []
          const lB = '{'
          const rB = '}'
          const result = replacedStr.replace(this.reg, target => {
            // .replace(/\s*/g,"") 去除空格
            const evalStr = target.replace(/[{}]/g, '')
            let temp = ''
            try {
              temp = eval(evalStr)
            } catch (error) {
              console.log('eval出错：', error)
            }
            return temp
          })
          // 这匹配{} 中加了@就可以把{}当做字符创处理
          const final = this.finalReplaceMethod(result)
          this.getFormatterResult = final.join('\n')
        },
        immediate: true
      },
      'formData.postFormatter': {
        handler(value) {
          let $ = {}
          const replacedStr = this.variableEval(value, $)
          const functionName = 'postBaselineAdd'
          const path = '/ops/baseline/add'
          const method = 'POST'
          const pathParams = []
          const queryParams = []
          const headerParams = []
          const bodyParams = ['alertId', 'alertOpen', 'alertTime']
          const result = replacedStr.replace(this.reg, target => {
            const evalStr = target.replace(/[{}]/g, '')
            return eval(evalStr)
          })
          const final = this.finalReplaceMethod(result)
          this.postFormatterResult = final.join('\n')
        },
        immediate: true
      },
      'formData.template': {
        handler(value) {
          this.templateResult = value
        },
        immediate: true
      }
    },
    methods: {
      variableEval(value, $) {
        return value.replace(this.variableReg, target => {
          const arr = target
            .replace(/\s*/g, '')
            .replace('$', '')
            .split('=')
          const evalStr = `$['${arr[0]}']=${arr[1]}`
          eval(evalStr)
          return ''
        })
      },
      finalReplaceMethod(str) {
        return str
          .replace('@', '')
          .split('\n')
          .filter(v => {
            return v.replace(/\s*/g, '') !== ''
          })
      },
      monacoEditorConfig() {
        // monaco.editor.create(this.$refs.getFormatter, {
        //   value: `console.log('hello')`,
        //   language: 'javascript',
        //   quickSuggestions: false,
        //   suggest: {
        //     filterGraceful: false
        //   },
        //   minimap: {
        //     enabled: false
        //   }
        // })
      },
      evalConst() {},
      onSubmit() {
        this.$refs.form.validate(valid => {
          if (!valid) {
            this.$message({
              message: '填写完整的配置项',
              type: 'warning'
            })
            return
          }
          this.$emit('emitGetListWithParams')
        })
      }
    }
  }
</script>

<style>
.container-textarea {
  display: flex;
  flex-direction: column;
}
.container-textarea > div {
  flex: 1;
}
/* vscode  */
/* .view-line {
  text-align: left;
} */
</style>
