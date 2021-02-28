<template>
  <div class="app-container">
    <h1 class="text-center">新增班级</h1>
    <el-form ref="form" v-loading="formLoading" :model="form" label-width="100px" :rules="rules">
      <el-row type="flex" class="row-bg" justify="space-between">
        <el-col :span="18" :offset="2" style="padding-top: 20px;">
          <el-form-item label="名称" prop="name" required>
            <el-input v-model="form.name" placeholder="请输入班级名称" />
          </el-form-item>
          <el-form-item>
            <el-row type="flex" class="row-bg" justify="end">
              <el-button @click="resetForm">重置</el-button>
              <el-button type="primary" @click="submitForm">提交</el-button>
            </el-row>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>
<style>
.el-select .el-input {
  width: 130px;
}

.input-with-select .el-input-group__prepend {
  background-color: #fff;
}

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}

.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
<script>
import { mapGetters, mapState, mapActions } from 'vuex'
import AV from 'leancloud-storage'

let editClass

export default {
  data() {
    return {
      form: {
        id: null,
        name: '',
        status: 1,
        date: new Date()
      },
      formLoading: false,
      rules: {
        name: [
          { required: true, message: '请输入班级名称', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {
    ...mapGetters('enumItem', [
      'enumFormat'
    ]),
    ...mapState('enumItem', {
      sexEnum: state => state.user.sexEnum,
      roleEnum: state => state.user.roleEnum,
      statusEnum: state => state.user.statusEnum,
      levelEnum: state => state.user.levelEnum
    })
  },
  created() {
    const id = this.$route.query.id
    const _this = this
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true

      const query = new AV.Query('Class')
      var p = this
      query.get(id).then((todo) => {
        p.form.name = todo.get('name')
        _this.formLoading = false
        editClass = todo
      })
    }
  },
  methods: {
    submitForm() {
      const _this = this
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log('this.$route.query.id', this.$route.query.id)
          const id = this.$route.query.id
          const Todo = AV.Object.extend('Lesson')

          var todo = new Todo()

          if (id && parseInt(id) !== 0) {
            todo = editClass
          }
          // 为属性赋值
          todo.set('name', this.form.name)
          todo.save().then((todo) => {
            // 成功保存之后，执行其他逻辑
            console.log(`保存成功。objectId：${todo.id}`)
            _this.$message({
              message: '成功',
              type: 'success'
            })
            _this.delCurrentView(_this).then(() => {
              _this.$router.push('/group/list')
            })
          }, (error) => {
            // 异常处理
            _this.formLoading = false
            console.log('save error', error)
          })
        } else {
          console.log('提交错误 !!')
          _this.formLoading = false
          return false
        }
      })
    },
    resetForm() {
      const lastId = this.form.id
      this.$refs['form'].resetFields()
      this.form = {
        id: null,
        name: '',
        status: 1
      }
      this.form.id = lastId
    },
    handleBeforeupload() {
      return this.$uploadhost.value
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log(file)
    },
    handleAvatarSuccess(res, file) {
      this.form.imageUrl = res['url']
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 PNG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    ...mapActions('tagsView', { delCurrentView: 'delCurrentView' })
  }
}
</script>
