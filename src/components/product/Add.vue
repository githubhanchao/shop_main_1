<template>
  <div class="goods-add">
    <el-steps :active="active" finish-status="success">
      <el-step :title="title1" description="基本信息"></el-step>
      <el-step :title="title2" description="商品图片"></el-step>
      <el-step :title="title3" description="商品内容"></el-step>
    </el-steps>
    <!-- 标签页 -->
    <el-tabs @tab-click="change" tab-position="left" v-model="activeName">
      <el-tab-pane name="basic" label="基本信息">
        <el-form label-width="80px" :model="addForm">
          <el-form-item label="商品名称">
            <el-input v-model="addForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格">
            <el-input v-model="addForm.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="商品重量">
            <el-input v-model="addForm.goods_weight"></el-input>
          </el-form-item>
          <el-form-item label="商品数量">
            <el-input v-model="addForm.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类">
            <el-cascader :options="options" v-model="addForm.goods_cat" :props="props"></el-cascader>
          </el-form-item>
          <el-form-item label="是否促销">
            <el-radio v-model="addForm.is_promote" :label="true">是</el-radio>
            <el-radio v-model="addForm.is_promote" :label="false">否</el-radio>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="next(2)">下一步</el-button>
          </el-form-item>
        </el-form>
      </el-tab-pane>
      <el-tab-pane name="pic" label="商品图片">
        <el-upload
          class="upload-demo"
          action="http://localhost:8888/api/private/v1/upload"
          list-type="picture"
          :headers="headers"
          :on-success="handerSuccess"
        >
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
        <el-button type="primary" @click="next(3)">下一步</el-button>
      </el-tab-pane>
      <el-tab-pane name="content" label="商品内容">
        <quill-editor v-model="addForm.goods_introduce"></quill-editor>
        <el-button type="primary" @click="addGood">添加</el-button>
        <!-- vue-quill-editor的使用
        1. yarn add vue-quill-editor
        2. // 导入vue-quill-editor
        import VueQuillEditor from 'vue-quill-editor'
        // 导入样式文件
        import 'quill/dist/quill.core.css'
        import 'quill/dist/quill.snow.css'
        import 'quill/dist/quill.bubble.css'
        Vue.use(VueQuillEditor)
        3. 在页面中使用富文本编辑器-->
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  data() {
    return {
      active: 0,
      activeName: 'basic',
      title1: '进行中',
      title2: '步骤2',
      title3: '步骤3',
      addForm: {
        goods_name: '',
        goods_price: '',
        goods_weight: '',
        goods_number: '',
        goods_cat: [],
        goods_introduce: '',
        pics: [],
        is_promote: false
      },
      options: [],
      props: {
        children: 'children',
        value: 'cat_id',
        label: 'cat_name'
      },
      headers: {
        Authorization: localStorage.getItem('token')
      }
    }
  },
  methods: {
    change(tab) {
      if (this.active === 0) {
        this.title1 = '进行中'
        this.title2 = '步骤2'
        this.title3 = '步骤3'
      } else if (this.active === 1) {
        this.title1 = '已完成'
        this.title2 = '进行中'
        this.title3 = '步骤3'
      } else if (this.active === 2) {
        this.title1 = '已完成'
        this.title2 = '已完成'
        this.title3 = '步骤3'
      }
    },
    next(step) {
      if (step === 2) {
        this.active = 1
        this.title1 = '已完成'
        this.title2 = '进行中'
        this.title3 = '步骤3'
        this.activeName = 'pic'
      } else {
        this.active = 2
        this.title1 = '已完成'
        this.title2 = '已完成'
        this.title3 = '进行中'
        this.activeName = 'content'
      }
    },
    handerSuccess(res, file, fileList) {
      if (res.meta.status === 200) {
        this.addForm.pics.push({
          pic: res.data.tmp_path
        })
      }
    },
    async addGood() {
      let res = await this.axios.post('goods', {
        ...this.addForm,
        goods_cat: this.addForm.goods_cat.join()
      })
      if (res.meta.status === 201) {
        this.$router.push('/goods')
      }
    }
  },
  async created() {
    let res = await this.axios.get('categories?type=3')
    if (res.meta.status === 200) {
      this.options = res.data
    }
  }
}
</script>

<style lang="less" scoped>
.quill-editor {
  background-color: #fff;
  /deep/ .ql-editor {
    height: 200px;
  }
}
</style>
