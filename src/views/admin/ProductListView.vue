<template>
  <main>
    <div id="app">
      <div class="container">
        <div class="text-end mt-4">
          <button class="btn btn-primary" @click.prevent="openModal('new')">
            建立新的產品
          </button>
        </div>
        <table class="table table-hover mt-4">
          <thead>
            <tr>
              <th width="120">
                分類
              </th>
              <th>產品名稱</th>
              <th width="120">
                原價
              </th>
              <th width="120">
                售價
              </th>
              <th width="100">
                是否啟用
              </th>
              <th width="120">
                編輯
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in products" :key="item.id">
              <td>{{ item.category }}</td>
              <td>{{ item.title }}</td>
              <td class="text-end">{{ item.origin_price }}</td>
              <td class="text-end">{{ item.price }}</td>
              <td>
                <span class="text-success" v-if="item.is_enabled">啟用</span>
                <span class="text-danger" v-else>未啟用</span>
              </td>
              <td>
                <div class="btn-group">
                  <button type="button" class="btn btn-outline-primary btn-sm" @click.prevent="openModal('edit', item)">
                    編輯
                  </button>
                  <button type="button" class="btn btn-outline-danger btn-sm" @click.prevent="openModal('delete', item)">
                    刪除
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- ProductModal -->
      <Product-Modal :temp-Product="tempProduct" :updateProduct="updateProduct" ref="pModal"></Product-Modal>
      <!-- 定義ref用於呼叫元件中的方法 -->
      <!-- 刪除產品 -->
      <!-- <Del-Modal :temp-Product="tempProduct" :delProduct="delProduct" @send-api="delProduct" ref="dModal"></Del-Modal> -->
      <!-- Modal -->
      <!-- pagination -->
      <div class="container">
        <PaginationItem :pages="pages" :get-products="getProducts"></PaginationItem>
      </div>
    </div>
  </main>
</template>

<script>
/* eslint-disable no-unused-vars */
//import { RouterView } from 'vue-router';
import axios from 'axios';
const { VITE_URL, VITE_PATH } = import.meta.env

import ProductModal from '@/components/ProductModal.vue';
import DelModal from '@/components/DelModal.vue';
import PaginationItem from '@/components/PaginationItem.vue';
import { createLogger } from 'vite';

export default {
  data() {
    return {
      products: [],
      isNew: false, //表示當前 Modal 是新增或編輯的判斷依據 
      tempProduct: {
        imagesUrl: [],
      },
      pages: {},
      productModal: null,
      delProductModal: null,
    };
  },

  methods: {
    async checkAdmin() {
      try {
        // 取得 token
        const token = document.cookie.replace(/(?:(?:^|.*;\s*)leleToken\s*=\s*([^;]*).*$)|^.*$/, '$1');
        //預設帶入token
        axios.defaults.headers.common['Authorization'] = token;

        // 發送 POST 請求，使用 await 等待完成
        await this.axios.post(`${VITE_URL}V2/api/user/check`);

        // POST 請求成功，執行下一步操作
        this.getProducts();
      } catch (err) {
        // POST 請求失敗，處理錯誤
        alert(err.response?.data?.message || 'Error occurred.');
        this.$router.push({ name: 'LoginView' });
      }
    },
    getProducts(page = 1) { //取得產品，products 是代表要有分頁的，all沒有
      this.axios.get(`${VITE_URL}V2/api/${VITE_PATH}/admin/products?page=${page}`)
        .then((res) => {
          //存資料
          this.products = res.data.products;
          this.pages = res.data.pagination;
          console.log(res);
        })
        .catch((err) => {
          alert(err.response.data.message);
        })
    },
    openProduct(item) {
      this.tempProduct = item;
    },
    openModal(status, item) { //打開編輯視窗，status 判斷目前點擊的是 新增/編輯/刪除 按鈕；item 代表當前點擊的產品資料
      if (status === 'new') {
        //點擊新增btn，清空當前產品內容，開啟productModal
        this.tempProduct = {
          imagesUrl: [],
        };
        this.isNew = true;
        //this.productModal.show();
        this.$refs.pModal.openModal();
      } else if (status === 'edit') {
        //點擊新增btn，將當前產品內容傳入，目的為串接刪除 API 需要取得產品的 id，開啟productModal
        this.tempProduct = { ...item };
        //如果不是陣列就補陣列以確保不論是否有圖片都能做新增圖片的行為(前面有預設tempProduct.imageUrl是一個陣列才顯示新增刪除按鈕)
        if (!Array.isArray(this.tempProduct.imagesUrl)) {
          this.tempProduct.imagesUrl = []
        }
        this.isNew = false;
        //this.productModal.show();
        this.$refs.pModal.openModal();
      } else if (status === 'delete') {
        this.tempProduct = { ...item };
        //this.delProductModal.show();
        this.$refs.dModal.openModal();
      }
    },
    updateProduct() {
      if (this.isNew) { //新增產品
        this.axios.post(`${VITE_URL}V2/api/${VITE_PATH}/admin/product`, { "data": this.tempProduct })
          .then((res) => {
            alert("產品新增成功");
            this.getProducts();
            //this.productModal.hide();
            this.$refs.pModal.closeModal();
          })
          .catch((err) => {
            alert(err.response.data.message);
          })
      } else { //修改產品
        this.axios.put(`${VITE_URL}V2/api/${VITE_PATH}/admin/product/${this.tempProduct.id}`, { "data": this.tempProduct })
          .then((res) => {
            alert("產品更新成功");
            //this.productModal.hide();
            this.$refs.pModal.closeModal();
            this.getProducts();
          })
          .catch((err) => {
            alert(err.response.data.message);
          })
      }
    },
    delProduct() {
      //console.log(this.tempProduct.id);
      this.axios.delete(`${VITE_URL}V2/api/${VITE_PATH}/admin/product/${this.tempProduct.id}`)
        .then((res) => {
          alert(res.data.message);
          this.$refs.dModal.closeModal();
          this.getProducts();
        })
        .catch((err) => {
          alert(err.response.data.message);
        })
    },
    clearInput() { //取消新增產品時，清空輸入框和圖檔
      this.tempProduct.imagesUrl = [];
      this.tempProduct.imagesUrl.push('');
    },
  },

  mounted() {
    this.checkAdmin()
    //產品 modal
    this.productModal = new this.$bs.Modal( //建立 modal 實體
      document.getElementById("productModal"),
      {
        keyboard: false, //禁止 user 透過 Esc 按鍵關閉
        backdrop: 'static' //禁止 user 點擊 modal 以外的地方關閉視窗
      }
    );
    this.delProductModal = new this.$bs.Modal(
      document.getElementById("delProductModal"),
      {
        keyboard: false,
        backdrop: 'static'
      }
    )
  },
  components: {
    PaginationItem,
    ProductModal,
  }

}

</script>
