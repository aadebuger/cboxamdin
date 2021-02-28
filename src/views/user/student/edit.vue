<template>
  <div class="app-container">
    <h1 class="text-center">新增学员</h1>
    <el-form ref="form" v-loading="formLoading" :model="form" label-width="100px" :rules="rules">
      <el-col :span="15" :offset="4" style="padding-top: 20px;">
        <el-row type="flex" class="row-bg" justify="space-between">
          <el-col :span="12">
            <el-form-item label="用户名：" prop="userName" hidden>
              <el-input v-model="form.userName" />
            </el-form-item>
            <!--
            <el-form-item label="密码："  required>
              <el-input v-model="form.password"></el-input>
            </el-form-item>
            -->
            <el-form-item label="姓名：" prop="realName" required>
              <el-input v-model="form.realName" placeholder="请输入真实姓名" />
            </el-form-item>
            <el-form-item label="手机号码：" required>
              <el-input v-model="form.phone" type="text" placeholder="请输入手机号" maxlength="11" show-word-limit />
            </el-form-item>
            <el-form-item label="身份证号码：" prop="idcard">
              <el-input v-model="form.idcard" placeholder="请输入证件号码" maxlength="18" />
            </el-form-item>
            <el-form-item label="士兵证号码：" prop="idcard">
              <el-input v-model="form.soldier" placeholder="请输入证件号码" maxlength="7" />
            </el-form-item>

            <el-form-item label="班级：" prop="group" required>
              <el-select v-model="form.group" placeholder="班级">
                <el-option v-for="item in levelEnum" :key="item.key" :value="item.key" :label="item.value" />
              </el-select>
            </el-form-item>
            <el-col :span="16">
              <el-form-item label="箱号：">
                <el-input v-model="form.boxNumber" type="text" placeholder="请输入箱号" maxlength="2" show-word-limit />
              </el-form-item>
            </el-col>
          </el-col>
          <el-form-item label="照片" prop="imageUrl">
            <el-upload
              class="avatar-uploader"
              :action="handleBeforeupload()"
              :show-file-list="false"
              :on-success="handleAvatarSuccess"
              :before-upload="beforeAvatarUpload"
            >
              <img v-if="form.imageUrl" :src="form.imageUrl" class="avatar"> <i
                v-else
                class="el-icon-plus avatar-uploader-icon"
              />
            </el-upload>
          </el-form-item>
        </el-row>
        <!--
        <el-form-item label="状态：" required>
          <el-select v-model="form.status" placeholder="状态">
            <el-option v-for="item in statusEnum" :key="item.key" :value="item.key" :label="item.value"></el-option>
          </el-select>
        </el-form-item>
        -->
        <el-form-item label="手机型号：">
          <el-row type="flex" class="row-bg" justify="space-between">
            <el-col :span="18">
              <el-input v-model="form.deviceid" />
            </el-col>
            <el-button type="primary" @click="onSearch">获取手机型号</el-button>
          </el-row>
        </el-form-item>
        <el-form-item>
          <el-row type="flex" class="row-bg" justify="end">
            <el-button @click="resetForm">重置</el-button>
            <el-button type="primary" @click="submitForm">提交</el-button>
          </el-row>
        </el-form-item>
      </el-col>
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

var editstudent
export default {
  data() {
    return {
      form: {
        id: null,
        userName: '',
        password: '',
        realName: '',
        role: 1,
        status: 1,
        age: '',
        sex: '',
        birthDay: null,
        phone: null,
        group: 1,
        imageUrl: '',
        deviceid: '',
        boxNumber: 1
      },
      formLoading: false,
      rules: {
        userName: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        realName: [
          { required: true, message: '请输入真实姓名', trigger: 'blur' }
        ],
        group: [
          { required: true, message: '请选择班级', trigger: 'change' }
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

      const query = new AV.Query('Student')
      var p = this
      query.get(id).then((todo) => {
        // todo 就是 objectId 为 582570f38ac247004f39c24b 的 Todo 实例
        console.log('Student', todo)
        console.log('department', todo.get('department'))
        p.form.userName = todo.get('name')
        // p.form.deviceid = todo.get('androidid')
        p.form.imageUrl = todo.get('imageurl')
        p.form.group = todo.get('group')
        p.form.boxNumber = todo.get('boxNumber')
        _this.formLoading = false
        editstudent = todo
      })
    }
  },
  methods: {
    onSearch() {
      console.log('onSearch')
      var p = this
      var query = new AV.Query('Androiddevicenew')
      query.first().then(function(device) {
        // students 是包含满足条件的 Student 对象的数组
        if (typeof device === 'undefined') {
          p.$message({
            message: '未接手机,请连接手机后再试',
            type: 'fail'
          })
          return
        }

        console.log(`students=`, device)
        console.log('name', device.get('name'))
        p.studentForm.deviceid = device.get('serial')
        p.studentForm.model = device.get('name')
      }).catch((error) => {
        console.error(error)
      })
    },
    submitForm() {
      const _this = this
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log('this.$route.query.id', this.$route.query.id)
          const id = this.$route.query.id
          const Todo = AV.Object.extend('Student')

          var todo = new Todo()

          if (id && parseInt(id) !== 0) {
            todo = editstudent
          }
          // 为属性赋值
          todo.set('name', this.form.userName)
          todo.set('realname', this.form.realName)
          todo.set('group', this.form.group)
          //          todo.set('androidid', this.form.deviceid)
          //          todo.set('photo', this.form.photo)
          todo.set('imageurl', this.form.imageUrl)
          todo.set('boxNumber', this.form.boxNumber)
          //          todo.set('department', this.form.depart)
          // 将对象保存到云端
          todo.save().then((todo) => {
            // 成功保存之后，执行其他逻辑
            console.log(`保存成功。objectId：${todo.id}`)
            _this.$message({
              message: '成功',
              type: 'success'
            })
            _this.delCurrentView(_this).then(() => {
              _this.$router.push('/user/student/list')
            })
          }, (error) => {
            // 异常处理
            _this.formLoading = false
            console.log('save error', error)
          })
        } else {
          console.log('error submit!!')
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
        userName: '',
        password: '',
        realName: '',
        role: 1,
        status: 1,
        age: '',
        sex: '',
        birthDay: null,
        phone: null,
        group: null,
        imageUrl: '',
        deviceid: ''
      }
      this.form.id = lastId
    },
    handleBeforeupload() {
      return this.$uploadhost.value
      //  return "http://localhost:7000/1.1/filesformminio/test.png"
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log(file)
    },
    handleExceed(files, fileList) {
      this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },
    handleAvatarSuccess(res, file) {
      // this.imageUrl = URL.createObjectURL(file.raw);
      console.log('res=', res)
      console.log('file=', file)
      // this.imageUrl='http://localhost:9000/boxhr/2060S.png'
      // this.imageUrl = URL.createObjectURL(file.raw);
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
