<template>
  <div>
    <el-button type="info" @click="logout">退出</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    logout() {
      //清空 sessionStorage 中的 token
      window.sessionStorage.clear()
      //通过编程式导航重定向到登录页
      this.$router.push('/login')
    }
  }
}
</script>

<style>

</style>