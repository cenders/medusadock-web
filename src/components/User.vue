<template>
<div>
  <sign-up :visible.sync="signUpVisible"></sign-up>
  <el-dialog :visible="visible" width="30%" @update:visible="updateVisible">

    <div v-if="$store.state.user">
      <table>
        <tr><td width="150px">User ID:</td><td v-text="user.id"></td></tr>
        <tr><td>User Name:</td><td v-text="user.username"></td></tr>
        <tr><td>Email:</td><td v-text="user.email"></td></tr>
      </table>
      <div>
        <div style="float: right" v-if="user">
          <el-button class="grey-button" type="text" @click.native="$store.commit('logout')">Logout</el-button>
        </div>
        <div style="clear: both"></div>
      </div>
    </div>

    <div v-if="!$store.state.user">
      <el-form @keyup.enter.native="submit" :model="form" label-width="120px" label-position="left">
        <el-form-item label="User Name">
          <el-input v-model="form.username" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="Password">
          <el-input type="password" v-model="form.password" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <div style="float: left">
          <el-button @click="updateVisible(false)">Cancel</el-button>
          <el-button type="warning" @click="submit">Login</el-button>
        </div>
        <div style="float: right">
          <a class="jn-button" @click="updateVisible(false);signUpVisible=true;"> Sign Up</a>
        </div>
        <div style="clear: both"></div>

      </div>
    </div>
  </el-dialog>
</div>
</template>

<script>
import axios from 'axios'
import SignUp from '@/components/SignUp'

export default {
  name: 'user',

  data () {
    return {
      signUpVisible: false,
      form: {
        username: '',
        password: ''
      }
    }
  },

  computed: {
    user () {
      return this.$store.state.user
    }
  },

  components: {
    SignUp
  },

  props: ['visible'],

  methods: {
    updateVisible (val) {
      this.$emit('update:visible', val)
    },

    submit () {
      let v = this
      let formData = new FormData()
      for (let i in v.form) {
        formData.append(i, v.form[i])
      }

      axios({
        method: 'post',
        url: v.$config.HOST + '/medusadock/actions/login.php',
        data: formData,
        config: {headers: {'Content-Type': 'multipart/form-data'}}
      }).then(response => {
        var r = response.data
        if (r.status === 1) {
          this.$store.commit('login', r.user)
//          v.updateVisible(false)
        } else {
          alert('Login failed! ' + r.msg)
        }
//        window.location.href = '#/' + 'TaskManagement'
      }).catch(() => {
        alert('Login failed!')
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dialog {
width: 100%;
height: 100%;
background: rgba(0,0,0,.8);
}

.loginPage {
position: absolute;
top: 50%;
left: 50%;
margin-top: -150px;
margin-left: -175px;
width: 350px;
min-height: 300px;
padding: 30px 20px 20px;
border-radius: 8px;
box-sizing: border-box;
background-color: #fff;
}

.loginPage p {
color: red;
text-align: left;
}
</style>
