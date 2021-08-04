<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert title="添加商品信息" type="info" center>
      </el-alert>
      <el-steps :active="activeIndex - 0" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-position="top">
        <el-tabs :tab-position="'left'" v-model="activeIndex" :before-leave="beforeTabLeave" @tab-click="handleTabClicked">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat" :options="cateList" :props="cascaderProps" @change="handleCascaderChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="item in manyParamList" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="param" v-for="(param,i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyParamList" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload :action="uploadURL" :headers="uploadHeader" :on-preview="handlePreview" :on-remove="handleRemove" :on-success="handleSuccess" list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加商品 -->
            <el-button type="primary" class="btnAdd" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览dialog -->
    <el-dialog title="图片预览" :visible.sync="previewVisiable" width="50%">
      <img :src="previewUrl" class="previewImg">
    </el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data () {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        attrs: [],
        pics: [],
        goods_introduce: ''
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      cateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      manyParamList: [],
      onlyParamList: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      uploadHeader: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewVisiable: false,
      previewUrl: ''
    }
  },
  created () {
    this.queryCateList()
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },
  methods: {
    async queryCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类数据失败')
      }
      this.cateList = res.data
    },
    handleCascaderChange () {
      console.log(this.addForm.goods_cat)
    },
    beforeTabLeave (activeName, odlActiveName) {
      // 未选中商品分类阻止Tab标签跳转
      if (odlActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async handleTabClicked () {
      if (this.activeIndex === '1') {
        this.manyParamList = await this.queryParams('many')
      }
      if (this.activeIndex === '2') {
        this.onlyParamList = await this.queryParams('only')
      }
    },
    async queryParams (selType) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: selType } })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.errror('查询属性失败！')
      }
      if (selType === 'many') {
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        })
      }
      return res.data
    },
    handlePreview (file) {
      console.log(file)
      this.previewUrl = file.response.data.url
      this.previewVisiable = true
    },
    handleRemove (file, fileList) {
      console.log(file)
      const index = this.addForm.pics.findIndex(item => item.pic === file.response.data.temp_path)
      this.addForm.pics.splice(index, 1)
      console.log(this.addForm.pics)
    },
    handleSuccess (response, file, fileList) {
      console.log(response)
      const value = { pic: response.data.tmp_path }
      this.addForm.pics.push(value)
      console.log(this.addForm.pics)
    },
    addGoods () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          this.$message.error('请填入必要参数')
          return
        }
        const addFormObj = _.cloneDeep(this.addForm)
        addFormObj.goods_cat = addFormObj.goods_cat.join(',')

        this.manyParamList.forEach(item => {
          const value = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(value)
        })

        this.onlyParamList.forEach(item => {
          const value = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals
          }
          this.addForm.attrs.push(value)
        })
        addFormObj.attrs = this.addForm.attrs

        // 发起请求添加商品
        // 商品名称必须是唯一的
        const { data: res } = await this.$http.post('goods', addFormObj)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败！')
        this.$message.success('添加商品成功!')
        this.$router.push('/goods')
      })
    }
  }
}
</script>
<style lang="less" scoped>
.el-steps {
  margin: 15px 0px;
}
.el-checkbox {
  margin: 0px 15px 0px 0px !important ;
}
.previewImg {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>
