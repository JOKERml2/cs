<template>
  <Row :span="24" type="flex" justify="center" align="middle">
    <Col>
      <Card style="margin-top:150px;min-width:400px;text-align:center">
        <Form
          ref="form"
          :model="formInline"
          :rules="ruleInline"
          method="post"
          :disabled="submitting"
          @submit.native.prevent
        >
          <h1>Please login</h1>
          <Divider />
          <FormItem prop="user">
            <Input type="text" v-model="formInline.user" placeholder="Username" name="name">
              <Icon type="ios-person-outline" slot="prepend"></Icon>
            </Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="password" v-model="formInline.password" placeholder="Password">
              <Icon type="ios-lock-outline" slot="prepend"></Icon>
            </Input>
          </FormItem>
<!--          <FormItem prop="role">-->
<!--            <input type="radio" id="cus" value="cus" v-model="formInline.role" />-->
<!--            <label for="cus">Customer/User</label>-->
<!--            <input type="radio" id="sys" value="sys" v-model="formInline.role" />-->
<!--            <label for="sys">System/Admin</label>-->
<!--          </FormItem>-->
          <FormItem>
            <Button type="primary" @click="handleSubmit('form', 'login')">Log In</Button>
            <Button @click="handleSubmit('form', 'signup')">Sign Up</Button>
          </FormItem>
        </Form>
      </Card>
    </Col>
  </Row>
</template>
<script>
import axios from 'axios'

export default {
  components: {
    axios
  },
  data () {
    return {
      choose_role: 'sys',
      formInline: {
        user: '',
        password: '',
        role: ''
      },
      submitting: false,
      ruleInline: {
        user: [
          { required: true, message: 'Please fill in the user name', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Please fill in the password.', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    createNew () {
      window.location = '/profile'
    },
    handleSubmit (name, action) {
      // if (action === 'signup') {
      //   if (this.choose_role === 'sys') {
      //     this.formInline.role = 'sys';
      //     this.choose_role = 'cus';
      //     console.log('0')
      //     console.log(this.choose_role)
      //   }
      //   else {
      //     this.formInline.role = 'cus';
      //     this.choose_role = 'sys';
      //     console.log('1');
      //     console.log(this.choose_role)
      //   }
      // }
      // else {
      //   this.formInline.role = 'anyone';
      // }
      this.$refs[name].validate((valid) => {
        if (valid) {
          let usrURL = 'http://localhost:9191/login?name=' + this.formInline.user + '&password=' + this.formInline.password + '&role=' + this.formInline.role + '&action=' + action
          axios.post(
            usrURL
          ).then((json) => {
            // debugger
            this.$Notice.success({
              title: 'Success!',
              desc: action === 'login' ? 'User logged in.' : 'User created.'
            })
            this.formInline.role = json.data.role
            this.submitting = true
            this.$http.post('/checkLogin',
              'username=' + this.formInline.user + '&password=' + this.formInline.password,
              {headers: {'Content-Type': 'application/x-www-form-urlencoded'}}
            ).then((response) => {
              if (response.data.code === 200) {
                // window.location = response.data.profile ? '/index?_uuid=' + response.data.uuid + '&_user=' + this.formInline.user : '/profile?_uuid=' + response.data.uuid + '&_user=' + this.formInline.user
                // window.location = '/index?_uuid=' + response.data.uuid + '&_user=' + this.formInline.user
                const data = {
                  uuid: response.data.uuid,
                  user: this.formInline.user
                }
                console.log(response.data)
                const queryParams = new URLSearchParams(data).toString()
                console.log(this.formInline.role)
                if (this.formInline.role === 'sys') {
                  window.location = `/Instructions_user.html?${queryParams}`
                } else if (this.formInline.role === 'cus') {
                  window.location = `/Instructions_cus.html?${queryParams}`
                }
              } else {
                this.$Message.error({
                  content: response.data.msg,
                  duration: 10,
                  closable: true
                })
                this.submitting = false
              }
            }).catch((e) => {
              console.log(e)
              this.submitting = false
              this.$Message.error('Something wrong when sending data!')
            })
          }).catch((error) => {
            // debugger
            console.log(error)
            this.$Message.error('Something wrong when sending data!')
          })
        }
      })
    }
  }
}
</script>
