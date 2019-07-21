<template>
	<div class="page">
		<div>
			<p class="title">Switch API</p>
			<el-form class="co_form_width" :model="form" ref="form" label-width="100px" size="small">
				<el-form-item label="SwaggerUrl：" prop="url">
					<el-input placeholder="请输入获取接口URL" size="medium"
										v-model.trim="form.url"></el-input>
				</el-form-item>
				<el-form-item label="项目地址：" prop="path">
					<el-input placeholder="请输入项目地址" size="medium"
										v-model.trim="form.path"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleRun()">运行</el-button>
				</el-form-item>
			</el-form>
		</div>
	</div>
</template>

<script>
import fs from 'fs'
// import path from 'path'
// import { remote } from 'electron'
// import API from '../../../../static/swagger'

export default {
  data () {
    return {
      form: {
        url: '',
        path: ''
      }
    }
  },
  methods: {
    handleRun () {
      this.getAPI()
      localStorage.setItem('switchApi', JSON.stringify(this.form))
      // 存储到localStorage
    },
    async getAPI () {
      try {
        console.log(this.form)
        let res = await this.$get(this.form.url)
        // let res = await this.$get('http://10.100.172.6:9101/v2/api-docs')
        let resultAPI = {}
        let ObjectAPI = res.paths
        // let ObjectAPI = API.paths
        Object.keys(ObjectAPI).map((v, k) => {
          let content = ObjectAPI[v]
          let url = v// 请求url
          let method = Object.keys(content)[0] // 获取请求方法
          let tags = content[method].tags// 获取tags
          let lastName = v.match(/{?\w+}?$/)[0].replace(/[{}]/g, '')
          // let methodName = `${lastName}Using${method.replace(/^(\w)/, ($1) => $1.toUpperCase())}`
          let methodName = `${lastName}-${method}`
          if (resultAPI[tags]) {
            Object.assign(resultAPI[tags], {
              [methodName]: {
                url,
                method
              }
            })
          } else {
            resultAPI[tags] = {
              [methodName]: {
                url,
                method
              }
            }
          }
        })
        console.log(resultAPI)
        // let str = `export default ${JSON.stringify(resultAPI, null, '\t').replace(/"/g, '\'')}`
        let str = `export default ${JSON.stringify(resultAPI, null, '\t')}`
        this.writeApiFile(str)
      } catch (error) {
        this.$message.error(error.message)
      }
    },
    writeApiFile (resultAPI) {
      try {
        let inputFileState = fs.existsSync(`${this.form.path}`)
        if (!inputFileState) {
          this.$message.warning('输入的路径不存在,请重新输入')
        } else {
          let fileState = fs.existsSync(`${this.form.path}/src/interface/`)
          if (!fileState) fs.mkdirSync(`${this.form.path}/src/interface/`)
          fs.writeFileSync(`${this.form.path}/src/interface/apis.js`, resultAPI, {flag: 'w'})
          this.$message.success('文件已生成')
        }
      } catch (error) {
        this.$message.error(error.message)
      }
    }
  },
  mounted () {
    let form = localStorage.getItem('switchApi')
    console.log(form)
    if (form !== 'null') this.form = JSON.parse(form)
  },
  components: {}
}
</script>

<style scoped>
</style>
