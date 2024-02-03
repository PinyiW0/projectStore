<script>
//import { RouterLink, RouterView } from 'vue-router'
export default {
  mounted() {
    console.log(this.$bs)
  }
}
</script>

<template>
  <header>
    <div class="container">
      <HelloWorld msg="You did it!" />
      <nav>
        <RouterLink  class="me-3 my-3" to="/">Home</RouterLink>
        <RouterLink class="me-3 my-3" to="/about">About</RouterLink>
        <RouterLink class="me-3 my-3" to="/LoginView">AdminLogin</RouterLink>
        <RouterLink to="/productList">ProductList</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

