<template>
  <div id="app">
    <div class="container">
      <div class="row justify-content-center">
        <h1 class="h3 my-5 font-weight-normal text-center">
          請先登入
        </h1>
        <div class="col-8 col-md-6 col-xl-5">
          <form id="form" class="form-signin" @submit.prevent="login">
            <div class="form-floating mb-3">
              <input type="email" class="form-control" v-model="user.username" id="username"
                placeholder="name@example.com" required autofocus>
              <label for="username">Email address</label>
            </div>
            <div class="form-floating">
              <input type="password" class="form-control" v-model="user.password" id="password" placeholder="Password"
                required>
              <label for="password">Password</label>
            </div>
            <button class="btn btn-lg btn-success w-100 mt-3" type="submit">
              登入
            </button>
          </form>
        </div>
      </div>
      <p class="mt-5 mb-3 text-muted text-center">
        &copy; 2024 六角學院 VUE直播班 - 主線作業
      </p>
    </div>
  </div>
</template>


<script>
const { VITE_URL } = import.meta.env
export default {
  data() {
    return {
      user: {
        "username": '',
        "password": '',
      }
    };
  },
  methods: {
    login() {
      this.axios.post(`${VITE_URL}V2/admin/signin`, this.user)
        .then((res) => {
          if (!res) {
            alert('不得填入空資訊');
            return;
          }

          if (res.status === 200) {
            //使用解構方式取出 token 和有效時間資料
            const { token, expired } = res.data;
            //儲存在 cookie 裡面
            document.cookie = `leleToken=${token}; expires=${new Date(expired)};`;
            //顯示登入成功
            window.alert('登入成功');
            //切換分頁
            this.$router.push({ name: 'ProductListView' });
          } else {
            this.$message.error('登入失敗');
          }
        })
        .catch((err) => {
          alert(err.response.data.message);
        });
    },
  },
}
</script>
