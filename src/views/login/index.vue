<template>
  <el-card style="width:25% ;height:30% ;margin:0 auto; margin-top:10%; padding:3%">
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="账号">
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" >
        <el-input v-model="form.password" type="password" ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit"  style="margin:0">登录</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script>
  export default {
    data() {
      return {
        form: {
          username:'',
          password:'',
        }
      }
    },
    methods: {
      onSubmit() {
        console.log('submit!');
        this.$axios
          .post("/api/Authoize/Login?username="+this.form.username+"&userpwd="+this.form.password)
          .then((response)=>{
            console.log(response.data.data.userName)
            this.$router.push({path:'/home', query:{id:response.data.data.id, name:response.data.data.userName}})
          })
      }
    }
  }
</script>