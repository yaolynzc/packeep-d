<template>
  <el-container>
    <el-header>
      <el-menu
        :default-active="activeIndex"
        class="el-menu-demo"
        mode="horizontal"
        background-color="#FFAF40"
        text-color="#000"
        active-text-color="#fff">
        <el-menu-item index="1">处理中心</el-menu-item>
      </el-menu>
      <div class="user-dropdown-menu">
        <img style="width:32px;height:32px;margin-top:15px;" :src="avatar">
        <span>&nbsp;</span>
        <el-dropdown @command="dropDownClick">
          <span class="el-dropdown-link">{{profileForm.nickname}}<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="profile"><i class="el-icon-info">&nbsp;个人信息</i></el-dropdown-item>
            <el-dropdown-item command="modpwd"><i class="el-icon-edit-outline">&nbsp;修改密码</i></el-dropdown-item>
            <el-dropdown-item command="logout"><i class="el-icon-back">&nbsp;退出系统</i></el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </el-header>
    <el-main>
      <el-row>
        <el-col :span="5">
          <el-form
            :model="ruleForm"
            status-icon :rules="rules"
            ref="ruleForm"
            label-width="100px">
            <el-tag>包裹入库</el-tag>
            <el-form-item label="手机：" prop="tel">
              <el-autocomplete
                v-model="ruleForm.tel"
                suffix-icon="el-icon-phone"
                ref="inputUphone"
                :fetch-suggestions="querySearch"
                :trigger-on-focus="false"
                @keyup.enter.native="getFocus('inputUname')"
                @input="searchUphone"
                @select="handleTelSelect">
              </el-autocomplete>
            </el-form-item>
            <el-form-item label="姓名：" prop="name">
              <el-input
                v-model="ruleForm.name"
                suffix-icon="el-icon-news"
                ref="inputUname"
                @keyup.enter.native="getFocus('inputPcode')"
                auto-complete="true">
              </el-input>
            </el-form-item>
            <el-form-item label="位置编号：" prop="code">
              <el-input
                v-model="ruleForm.code"
                suffix-icon="el-icon-location"
                ref="inputPcode"
                @keyup.enter.native="submitForm('ruleForm')"
                auto-complete="true">
              </el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" icon="el-icon-check" @click="submitForm('ruleForm')">入库</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="19">
          <el-row>
            <el-col :span="24">
              <div class="search-params-block">
                手机尾号：
                <el-input
                  placeholder="请输入手机尾号"
                  :autofocus="true"
                  prefix-icon="el-icon-search"
                  v-model="packData.userphone"
                  clearable
                  @keyup.enter.native="search"
                  @clear="search">
                </el-input>
                <el-button type="primary" icon="el-icon-search" @click="search">搜索</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="deletePack">删除</el-button>
                <el-button type="success" icon="el-icon-phone" @click="dialBatch">批量电话</el-button>
              </div>
            </el-col>
            <el-col :span="24" style="margin-top: 15px">
              <el-tabs v-model="activeName" @tab-click="tabhandleClick">
              <el-tab-pane label="未签收" name="0">
                <el-row class="daytimeButtons">
                  <el-button size="mini" type="primary" plain @click="searchByTime(0)">今天</el-button>
                  <el-button size="mini" type="primary" plain @click="searchByTime(1)">昨天</el-button>
                  <el-button size="mini" type="primary" plain @click="searchByTime(6)">近七天</el-button>
                  <el-button size="mini" type="primary" plain @click="search">全部</el-button>
                </el-row>
                <el-table
                  v-loading="loading"
                  element-loading-text="加载中"
                  element-loading-spinner="el-icon-loading"
                  element-loading-background="rgba(0, 0, 0, 0.8)"
                  :data="packData.data"
                  @selection-change="handleSelChange"
                  style="width: 100%">
                  <el-table-column
                    type="selection"
                    width="50">
                  </el-table-column>
                  <el-table-column
                    label="位置编号"
                    prop="Poscode"
                    width="100">
                  </el-table-column>
                  <el-table-column
                    label="手机"
                    prop="Userphone"
                    width="150"
                    align="center">
                  </el-table-column>
                  <el-table-column
                    label="姓名"
                    prop="Username"
                    width="100"
                    align="center">
                  </el-table-column>
                  <el-table-column
                    label="入库时间"
                    prop="Intime"
                    width="200"
                    align="center"
                    :formatter="intimeFormatter">
                  </el-table-column>
                  <el-table-column
                    label="入库天数"
                    prop="Packday"
                    width="100"
                    align="center">
                  </el-table-column>
                  <el-table-column
                    label="电话次数"
                    prop="Havedial"
                    width="100"
                    align="center">
                  </el-table-column>
                  <el-table-column label="操作" align="center">
                    <template slot-scope="scope">
                      <el-button
                        icon="el-icon-edit"
                        size="mini"
                        type="primary"
                        @click="handleOutClick(scope.$index, scope.row)">签收</el-button>
                      <el-button
                        icon="el-icon-phone"
                        size="mini"
                        type="success"
                        @click="handleDialClick(scope.$index, scope.row)">电话</el-button>
                      <el-button
                        icon="el-icon-view"
                        size="mini"
                        type="warning"
                        @click="openCamera(scope.$index, scope.row)">拍照</el-button>
                    </template>
                  </el-table-column>
                </el-table>
                <div align="center">
                  <el-pagination
                    @current-change="handleCurrentChange"
                    v-show="packData.pagination.total"
                    background
                    layout="prev, pager, next"
                    :page-size="packData.pagination.size"
                    :total="packData.pagination.total">
                  </el-pagination>
                </div>
              </el-tab-pane>
              <el-tab-pane label="已签收" name="1">
                <el-row class="daytimeButtons">
                  <el-button size="mini" type="primary" plain @click="searchByTime(0)">今天</el-button>
                  <el-button size="mini" type="primary" plain @click="searchByTime(1)">昨天</el-button>
                  <el-button size="mini" type="primary" plain @click="searchByTime(6)">近七天</el-button>
                  <el-button size="mini" type="primary" plain @click="search">全部</el-button>
                </el-row>
                <el-table
                  v-loading="loading"
                  element-loading-text="加载中"
                  element-loading-spinner="el-icon-loading"
                  element-loading-background="rgba(0, 0, 0, 0.8)"
                  :data="packData.data"
                  style="width: 100%">
                  <el-table-column
                    label="位置编号"
                    prop="Poscode"
                    width="180">
                  </el-table-column>
                  <el-table-column
                    label="手机"
                    prop="Userphone"
                    width="180"
                    align="center">
                  </el-table-column>
                  <el-table-column
                    label="姓名"
                    prop="Username"
                    width="180"
                    align="center">
                  </el-table-column>
                  <el-table-column
                    label="签收照片"
                    prop="Havepic"
                    width="180"
                    align="center">
                    <template slot-scope="scope">
                      <img  :src="uploadPicPre + scope.row.Havepic" alt="" style="width: 80px;height: 60px">
                    </template>
                  </el-table-column>
                  <el-table-column
                    label="出库时间"
                    prop="Outtime"
                    align="center"
                    :formatter="outtimeFormatter">
                  </el-table-column>
                </el-table>
                <div align="center">
                  <el-pagination
                    @current-change="handleCurrentChange"
                    v-show="packData.pagination.total"
                    background
                    layout="prev, pager, next"
                    :page-size="packData.pagination.size"
                    :total="packData.pagination.total">
                  </el-pagination>
                </div>
              </el-tab-pane>
              </el-tabs>
            </el-col>
          </el-row>
          <el-dialog title="取件拍照" width="600px" :visible.sync="dialogCameraVisible" @close="closeCamera">
            <div>
              <video ref="video" width="560" height="420" autoplay></video>
              <canvas ref="canvas" width="560" height="420" style="display:none"></canvas>
              <div slot="footer" class="dialog-footer">
                <el-button icon="el-icon-view" type="warning" @click="snapCamera">{{snapCameraBtn}}</el-button>
                <el-button icon="el-icon-success" type="primary" @click="uploadPicClick">确定</el-button>
              </div>
            </div>
          </el-dialog>
          <el-dialog title="个人信息" width="500px" :visible.sync="dialogProfileVisible" @close="closeProfileDialog">
              <el-form :model="profileForm" ref="profileForm" status-icon :rules="profileRules">
                <el-form-item label="手机号：" :label-width="profileForm.itemWidth">
                  <span>{{profileForm.id}}</span>
                </el-form-item>
                <el-form-item label="用户名：" prop="username" :label-width="profileForm.itemWidth">
                  <el-input @keyup.enter.native="getFocus('inputNickname')" v-model="profileForm.username" :disabled="profileForm.inputDisable" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="昵称：" prop="nickname" :label-width="profileForm.itemWidth">
                  <el-input @keyup.enter.native="getFocus('inputAddress')" ref="inputNickname" v-model="profileForm.nickname" :disabled="profileForm.inputDisable" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="地址：" prop="address" style="margin-left:25px;">
                  <el-input style="width: 350px;" @keyup.enter.native="profileEditClick" ref="inputAddress" v-model="profileForm.address" :disabled="profileForm.inputDisable" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" icon="el-icon-edit" style="margin-left:50px;" @click="profileEditClick">{{profileEditText}}</el-button>
                </el-form-item>
              </el-form>
          </el-dialog>
          <el-dialog title="修改密码" width="450px" :visible.sync="dialogModPwdVisible">
            <el-form :model="modpwdForm" ref="modpwdForm" status-icon :rules="modpwdRules">
              <el-form-item label="新密码：" prop="pass" :label-width="modpwdForm.itemWidth">
                <el-input type="password" @keyup.enter.native="getFocus('inputCheckpass')" v-model="modpwdForm.pass" auto-complete="off"></el-input>
              </el-form-item>
              <el-form-item label="确认密码：" prop="checkpass" :label-width="modpwdForm.itemWidth">
                <el-input type="password" @keyup.enter.native="modpwdSubmitClick('modpwdForm')" ref="inputCheckpass" v-model="modpwdForm.checkpass" auto-complete="off"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" icon="el-icon-check" style="margin-left:50px;" @click="modpwdSubmitClick('modpwdForm')">确定</el-button>
              </el-form-item>
            </el-form>
          </el-dialog>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
// 引入百度语音RESTful跨域请求api
import BaiduAip from '@/utils/baidu_tts_cors.js'
import moment from 'moment'
import md5 from 'md5'
import menavatar from '@/assets/img/men.png'

export default {
  name: 'index',
  data () {
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'))
      } else {
        if (this.modpwdForm.checkpass !== '') {
          this.$refs.modpwdForm.validateField('checkpass')
        }
        callback()
      }
    }
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.modpwdForm.pass) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    return {
      uid: localStorage.getItem('uid') || sessionStorage.getItem('uid'),
      upass: localStorage.getItem('upass') || sessionStorage.getItem('upass'),
      bdtoken: '',
      avatar: menavatar,
      uploadPicPre: 'http://211.159.218.41/',
      userTelFill: [
        // { 'value': '153', 'name': 'chen' },
        // { 'value': '186', 'name': 'flame' }
      ],
      mulSelData: [],
      dialogCameraVisible: false,
      dialogProfileVisible: false,
      dialogModPwdVisible: false,
      dialogCameraPackID: '',
      snapCameraBtn: '拍照',
      activeName: '0',
      loading: null,
      ruleForm: {
        code: '',
        tel: '',
        name: ''
      },
      rules: {
        code: [
          {required: true, message: '请输入包裹位置编号', trigger: 'blur'},
          {pattern: /^[0-9]*$/, message: '请输入数字编号！', trigger: 'blur'}
        ],
        tel: [
          {required: true, message: '请输入手机号码', trigger: 'blur'},
          {min: 11, max: 11, message: '请输入11位手机号码', trigger: 'blur'},
          {pattern: /^[1-9]{1}\d{10}$/, message: '手机号格式不对！', trigger: 'blur'}
        ],
        name: []
      },
      profileForm: {
        id: '',
        username: '',
        nickname: '',
        address: '',
        itemWidth: '120px',
        inputDisable: true
      },
      profileRules: {
        username: [
          {required: true, message: '请输入用户名', trigger: 'blur'}
        ],
        nickname: [
          {required: true, message: '请输入用户昵称', trigger: 'blur'}
        ],
        address: [
          {required: true, message: '请输入用户地址', trigger: 'blur'}
        ]
      },
      profileEditText: '修改',
      modpwdForm: {
        pass: '',
        checkpass: '',
        itemWidth: '120px'
      },
      modpwdRules: {
        pass: [
          { required: true, validator: validatePass, trigger: 'blur' }
        ],
        checkpass: [
          { required: true, validator: validatePass2, trigger: 'blur' }
        ]
      },
      packData: {
        userphone: '',
        data: [],
        pagination: {
          total: 0,
          page: 1,
          size: 10
        }
      },
      input2: '',
      activeIndex: '1'
    }
  },
  created () {
    this.naviMethod()
  },
  methods: {
    naviMethod () {
      if (this.checkLogin()) {
        this.getProfile()
        this.search()
      } else {
        this.$router.push('/login')
      }
    },
    getProfile () {
      // console.log(this.uid)
      // 获取个人信息
      let obj = {
        pwd: this.upass
      }
      let that = this
      this.$http.get('http://211.159.218.41/api/user/' + this.uid, {params: obj})
        .then(function (res) {
          if (res.data.success) {
            that.profileForm.id = res.data.user.Id
            that.profileForm.username = res.data.user.Username
            that.profileForm.nickname = res.data.user.Nickname
            that.profileForm.address = res.data.user.Address
          }
        })
        .catch(function (err) {
          console.log(err.message)
        })

      // 获取百度语音合成token
      this.$http.get('http://211.159.218.41/api/config', {params: {'name': 'bdtoken'}})
        .then(function (res) {
          if (res.data.success) {
            // console.log(JSON.parse(res.data.bdtoken).access_token)
            // json字符串转json对象，获取百度语音合成token的access_token值
            that.bdtoken = JSON.parse(res.data.bdtoken).access_token
          }
        })
        .catch(function (err) {
          console.log(err.message)
        })
    },
    getCount (uphone, state, daytime) {
      // 总数、页码、列表数据先归零和清空
      this.packData.pagination.total = 0
      this.packData.pagination.page = 1
      this.packData.data = []
      // console.log(daytime)
      let obj = {
        uphone: uphone,
        state: state,
        daytime: daytime,
        uid: this.uid,
        rad: Math.random()
      }

      this.loading = true
      let that = this
      this.$http.get('http://211.159.218.41/api/pack', {params: obj})
        .then(function (res) {
          if (res.data.count > 0) {
            that.packData.pagination.total = res.data.count
            that.getList(uphone, state, daytime)
          } else {
            that.packData.data = []
            that.loading = false
          }
        })
        .catch(function (err) {
          console.log(err.message)
        })
    },
    getList (uphone, state, daytime) {
      let obj = {
        uphone: uphone,
        state: state,
        daytime: daytime,
        uid: this.uid,
        page: this.packData.pagination.page,
        size: this.packData.pagination.size,
        rad: Math.random()
      }

      let that = this
      this.$http.get('http://211.159.218.41/api/pack', {params: obj})
        .then(function (res) {
          that.loading = false
          let redt = res.data.dt
          if (redt && redt.length > 0) {
            for (let item of redt) {
              item.Packday = moment(new Date().getTime()).diff(moment(item.Intime), 'days')
            }
            that.packData.data = redt
          }

          if (uphone) {
            let redt = res.data.dt
            let text = '包裹位置编号'
            let audio = null

            // 拼接检索到的编码
            for (let n of redt) {
              text += n.Poscode
            }
            // 向百度发送语音合成请求
            audio = BaiduAip.btts({
              tex: text,
              tok: that.bdtoken,
              spd: 5,
              pit: 5,
              vol: 9,
              per: 0
            }, {
              volume: 0.3,
              autoDestory: true,
              timeout: 10000,
              hidden: true,
              onInit: function (htmlAudioElement) {
              },
              onSuccess: function (htmlAudioElement) {
                // 合成后，自动播放语音
                audio = htmlAudioElement
                audio.play()
              },
              onError: function (errorText) {
              },
              onTimeout: function () {
              }
            })
          }
        })
        .catch(function (err) {
          console.log(err.message)
        })
    },
    // 点击页码
    handleCurrentChange (val) {
      this.packData.pagination.page = val
      this.getList(this.packData.userphone, this.activeName, '')
    },
    // 搜索
    search () {
      this.getCount(this.packData.userphone, this.activeName, '')
    },
    searchByTime (day) {
      moment.locale('zh-cn')
      let daynow = moment()
      let daytime = daynow.subtract(day, 'days').format('YYYY-MM-DD')
      this.getCount('', this.activeName, daytime)
    },
    // 入库
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let obj = {
            pcode: this.ruleForm.code,
            uphone: this.ruleForm.tel,
            uname: this.ruleForm.name,
            uid: this.uid
          }

          let that = this
          this.$http.post('http://211.159.218.41/api/pack', this.$qs.stringify(obj))
            .then(function (res) {
              if (res.data.success) {
                that.$message({
                  message: '入库成功！',
                  type: 'success'
                })
                that.resetForm()
                that.getCount('', that.activeName, '')
              }
            })
            .catch(function (err) {
              console.log(err.message)
            })
        }
      })
    },
    resetForm () {
      this.getFocus('inputUphone')
      this.$refs['ruleForm'].resetFields()
      this.$refs['ruleForm'].clearValidate()
      this.userTelFill = []
    },
    // tab切换点击事件
    tabhandleClick (tab, event) {
      this.packData.data = []
      this.getCount(this.packData.userphone, this.activeName, '')
    },
    // 不拍照直接签收
    handleOutClick (index, row) {
      // console.log(index, row)
      let obj = {
        state: 1
      }

      let that = this
      this.$http.put('http://211.159.218.41/api/pack/' + row.Id, this.$qs.stringify(obj))
        .then(function (res) {
          if (res.data.success) {
            that.$message({
              message: '签收成功！',
              type: 'success'
            })
            // 关闭摄像头
            that.closeCamera()
            // 刷新页面
            that.getCount(that.packData.userphone, that.activeName, '')
          }
        })
        .catch(function (err) {
          console.log(err.message)
        })
    },
    // 拍照上传并完成签收
    uploadPicClick () {
      let picBase64 = this.$refs.canvas.toDataURL().substring(22)
      // console.log(picBase64)
      let obj = {
        state: 1,
        havepic: picBase64
      }
      let that = this
      this.$http.post('http://211.159.218.41/api/pack/' + this.dialogCameraPackID, this.$qs.stringify(obj))
        .then(function (res) {
          if (res.data.success) {
            that.dialogCameraVisible = false
            // 显示成功消息
            that.$message({
              message: '签收成功！',
              type: 'success'
            })
            that.getCount(that.packData.userphone, that.activeName, '')
          } else {
            // 显示失败消息
            that.$message({
              message: '照片保存失败！',
              type: 'error'
            })
          }
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    handleDialClick (index, row) {
      let obj = {
        uphone: row.Userphone,
        uname: row.Username,
        uid: this.uid
      }
      let dialRes = this.openFullScreenLoading()
      // console.log(dialRes)
      if (dialRes) {
        let that = this
        this.$http.post('http://211.159.218.41/api/phone', this.$qs.stringify(obj))
          .then(function (res) {
            if (res.data.success) {
              // 显示电话通知成功消息
              that.$message({
                message: '电话通知成功！',
                type: 'success'
              })
              // 电话通知成功后刷新页面
              let obj = {
                havedial: 1
              }
              that.$http.put('http://211.159.218.41/api/pack/' + row.Id, that.$qs.stringify(obj))
                .then(function (res) {
                  if (res.data.success) {}
                })
                .catch(function (err) {
                  console.log(err.message)
                })
            } else {
              // 显示电话通知失败消息
              that.$message({
                message: '通知失败！',
                type: 'error'
              })
            }
          })
          .catch(function (err) {
            console.log(err.message)
          })
      }
    },
    // 批量电话通知
    dialBatch () {
      let dialPack = this.mulSelData
      if (dialPack.length === 0) {
        this.$message.error('请勾选需要电话通知的包裹!')
      } else {
        this.$confirm('您确定给所勾选的包裹用户拨打电话?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let that = this
          for (let item of dialPack) {
            let obj = {
              uphone: item.Userphone,
              uname: item.Username,
              uid: this.uid
            }
            this.$http.post('http://211.159.218.41/api/phone', this.$qs.stringify(obj))
              .then(function (res) {
                if (res.data.success) {
                  // 电话通知成功后刷新页面
                  let obj = {
                    havedial: 1
                  }
                  that.$http.put('http://211.159.218.41/api/pack/' + item.Id, that.$qs.stringify(obj))
                    .then(function (res) {
                      if (res.data.success) {
                        // 显示电话通知成功消息
                        that.$notify({
                          title: item.Userphone,
                          message: '电话通知成功！',
                          position: 'bottom-right',
                          type: 'success'
                        })
                      }
                    })
                    .catch(function (err) {
                      console.log(err.message)
                    })
                } else {
                  // 显示电话通知失败消息
                  that.$notify.error({
                    title: item.Userphone,
                    message: '电话通知失败！',
                    position: 'bottom-right'
                  })
                }
              })
              .catch(function (err) {
                console.log(err.message)
              })
          }
          // 刷新页面
          this.openFullScreenLoading()
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消批量拨打电话！'
          })
        })
      }
    },
    // 删除入库记录
    deletePack () {
      let delPack = this.mulSelData
      if (delPack.length === 0) {
        this.$message.error('请勾选删除项!')
      } else {
        this.$confirm('您确定删除所勾选记录?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let that = this
          for (let item of delPack) {
            this.$http.delete('http://211.159.218.41/api/pack/' + item.Id)
              .then(function (res) {
                if (res.data.success) {
                  that.$message({
                    type: 'success',
                    message: '删除成功!',
                    onClose: () => {
                      that.search()
                    }
                  })
                }
              })
              .catch(function (error) {
                console.log(error)
                that.$message({
                  type: 'error',
                  message: '删除失败!'
                })
              })
          }
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      }
    },
    intimeFormatter (row, column) {
      return moment(row.Intime).format('YYYY-MM-DD HH:mm:ss')
    },
    outtimeFormatter (row, column) {
      return moment(row.Outtime).format('YYYY-MM-DD HH:mm:ss')
    },
    getFocus (inputname) {
      this.$refs[inputname].focus()
    },
    searchUphone () {
      if (this.ruleForm.tel.length > 2) {
        // console.log(this.ruleForm.tel.length)

        let obj = {
          uphone: this.ruleForm.tel,
          size: this.packData.pagination.size,
          rad: Math.random()
        }

        let that = this
        this.$http.get('http://211.159.218.41/api/phone', {params: obj})
          .then(function (res) {
            if (res.data.dt && res.data.dt.length > 0) {
              let redt = res.data.dt
              // 挑选userphone和username
              for (let n of redt) {
                n.value = n.Userphone
              }
              that.userTelFill = redt
            }
          })
          .catch(function (err) {
            console.log(err.message)
          })
      }
    },
    handleSelChange (val) {
      this.mulSelData = val
    },
    querySearch (queryString, cb) {
      // 调用 callback 返回建议列表的数据
      cb(this.userTelFill)
    },
    handleTelSelect (item) {
      this.ruleForm.name = item.Username

      // 根据手机号查询姓名，填充到姓名input
      // let that = this
      // this.$http.get('http://211.159.218.41/api/phone/' + this.ruleForm.tel)
      //   .then(function (res) {
      //     if (res.data.success) {
      //       that.ruleForm.name = res.data.pack.Username
      //     }
      //   })
      //   .catch(function (error) {
      //     console.log(error)
      //   })
    },
    // 打开摄像头
    openCamera (index, row) {
      this.snapCameraBtn = '拍照'
      this.dialogCameraPackID = row.Id
      this.dialogCameraVisible = true
      this.$nextTick(() => {
        let video = this.$refs['video']
        let videoObj = {'video': {width: 560, height: 420}}

        if (navigator.mediaDevices.getUserMedia || navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia) {
          // 调用用户媒体设备, 访问摄像头
          this.getUserMedia(videoObj,
            function (mediaStream) {
              video.srcObject = mediaStream
              // video.play()
              if (video.paused) {
                video.play()
              }
            },
            function (error) {
              console.log(`访问用户媒体设备失败${error.name}, ${error.message}`)
            }
          )
        } else {
          this.$message({
            type: 'info',
            message: '不支持访问用户媒体'
          })
        }
      })
    },
    // 拍照
    snapCamera () {
      this.$nextTick(() => {
        if (this.snapCameraBtn === '拍照') {
          this.snapCameraBtn = '重拍'
          // 捕获影像到canvas
          let canvas = this.$refs['canvas']
          canvas.getContext('2d').drawImage(this.$refs['video'], 0, 0, 560, 420)
          // 捕获后暂停摄像头
          this.$refs['video'].pause()
        } else {
          this.snapCameraBtn = '拍照'
          // 继续播放摄像头
          this.$refs['video'].play()
        }
      })
    },
    // 关闭摄像头
    closeCamera () {
      this.$nextTick(() => {
        this.$refs['video'].srcObject = null
      })
    },
    // 访问用户媒体设备的兼容方法
    getUserMedia (constraints, success, error) {
      if (navigator.mediaDevices.getUserMedia) {
        // 最新的标准API
        navigator.mediaDevices.getUserMedia(constraints).then(success).catch(error)
      } else if (navigator.webkitGetUserMedia) {
        // webkit核心浏览器
        navigator.webkitGetUserMedia(constraints, success, error)
      } else if (navigator.mozGetUserMedia) {
        // firfox浏览器
        navigator.mozGetUserMedia(constraints, success, error)
      } else if (navigator.getUserMedia) {
        // 旧版API
        navigator.getUserMedia(constraints, success, error)
      }
    },
    // 个人菜单点击事件
    dropDownClick (command) {
      // this.$message('click on item ' + command)
      let that = this
      switch (command) {
        case 'profile':
        {
          that.dialogProfileVisible = true
          break
        }
        case 'modpwd':
        {
          that.dialogModPwdVisible = true
          break
        }
        case 'logout':
        {
          // HTML5特性，清空所有本地存储或Session存储的全部变量
          localStorage.clear()
          sessionStorage.clear()
          that.$router.push('/login')
          break
        }
        default:
          break
      }
    },
    // 个人信息修改按钮点击事件
    profileEditClick () {
      if (this.profileEditText === '修改') {
        this.profileForm.inputDisable = false
        this.profileEditText = '确定'
      } else {
        this.submitProfile('profileForm')
      }
    },
    // 个人信息对话框关闭按钮点击事件
    closeProfileDialog () {
      this.profileForm.inputDisable = true
      this.profileEditText = '修改'
    },
    // 更新个人信息
    submitProfile (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let obj = {
            username: this.profileForm.username,
            nickname: this.profileForm.nickname,
            address: this.profileForm.address,
            rad: Math.random()
          }
          let that = this
          this.$http.put('http://211.159.218.41/api/user/' + this.profileForm.id, this.$qs.stringify(obj))
            .then(function (res) {
              if (res.data.success) {
                that.$message({
                  message: '修改成功！',
                  type: 'success'
                })
                that.profileEditText = '修改'
                that.profileForm.inputDisable = true
                that.dialogProfileVisible = false
              } else {
                that.$message({
                  message: '值未变化！',
                  type: 'info'
                })
              }
            })
            .catch(function (err) {
              console.log(err.message)
            })
        }
      })
    },
    // 修改密码
    modpwdSubmitClick (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let obj = {
            pwd: md5(this.modpwdForm.pass),
            rad: Math.random()
          }
          let that = this
          this.$http.put('http://211.159.218.41/api/user/' + this.uid, this.$qs.stringify(obj))
            .then(function (res) {
              if (res.data.success) {
                that.$message({
                  message: '修改成功！',
                  type: 'success'
                })
                that.dialogModPwdVisible = false
              } else {
                that.$message({
                  message: '值未变化！',
                  type: 'info'
                })
              }
            })
            .catch(function (err) {
              console.log(err.message)
            })
        } else {
          console.log('密码设置异常!')
          return false
        }
      })
    },
    // 满屏loading
    openFullScreenLoading () {
      let res = false
      const loading = this.$loading({
        lock: true,
        text: '接通中...',
        spinner: 'el-icon-phone',
        background: 'rgba(0, 0, 0, 0.7)'
      })
      let that = this
      res = setTimeout(() => {
        loading.close()
        // 刷新列表
        that.getCount(that.packData.userphone, that.activeName, '')
      }, 2000)
      return res
    },
    // 检测是否登录
    checkLogin () {
      let res = false
      let localUid = localStorage.getItem('uid')
      let sessionUid = sessionStorage.getItem('uid')
      // console.log(sessionUid + '/' + localUid)
      if (localUid || sessionUid) {
        res = true
      }
      return res
    }
  }
}
</script>

<style scoped>
  .el-tag {
    margin:0px 0px 15px 0px;
    font-size: 15px;
  }
  .el-form {
    padding-right: 20px;
  }
  .el-form-item {
    margin-left: -20px;
  }
  .el-form-item .el-input {
    width:auto;
  }
  .el-autocomplete {
    width:auto;
  }
  .search-params-block {
    float: left;
  }
  .search-params-block .el-input {
    width: 200px;
  }
  .el-pagination {
    margin-top: 20px;
  }
  .user-dropdown-menu {
    position:absolute;
    top: 10px;
    right: 0;
    width: 150px;
    heigth: 100%;
  }
  .el-dropdown {
    position: absolute;
    top:20px;
  }
  .el-dropdown-link {
    cursor: pointer;
    color: #ffffff;
  }
  .tab-tag-search {
    cursor:pointer;
  }
  .daytimeButtons {
    margin-bottom: 10px;
  }
</style>
