<template>
    <div class="app-container">
      <h1>熱量查詢系統</h1>
      <div class="search-container">
        <input v-model="searchQuery" placeholder="輸入食物名稱或俗名" />
        <button @click="searchFood">搜尋</button>
      </div>
      <div v-if="foods.length > 0" class="food-list">
        <div v-for="food in foods" :key="food._id" class="food-item">
          <h3>{{ food.樣品名稱 }}</h3>
          <p>俗名: {{ food.俗名 }}</p>
          <p>熱量: {{ food.修正熱量(kcal) }} 大卡</p>
        </div>
      </div>
      <div v-else>
        <p>查無資料</p>
      </div>
    </div>
  </template>
  
  <script>
import axios from "axios";

export default {
  data() {
    return {
      searchQuery: "", // 用戶輸入的查詢關鍵字
      foods: [], // 顯示查詢結果
    };
  },
  methods: {
    async searchFood() {
      console.log("Search Query:", this.searchQuery);  // 打印查詢的關鍵字
      if (this.searchQuery.trim()) {
        try {
          // 使用雲端後端 API URL
          const response = await axios.get("https://food-server-ycm2.onrender.com/api/search", {
            params: { query: this.searchQuery }, // 傳遞查詢參數
          });
          console.log(response);  // 查看是否獲得數據
          this.foods = response.data; // 更新查詢結果
        } catch (error) {
          console.error("搜尋錯誤:", error);
        }
      }
    },
  },
};
</script>
  
  <style scoped>
  /* 樣式可以根據需要進行自定義 */
  .app-container {
    font-family: Arial, sans-serif;
    padding: 20px;
    background-color: #f4f4f9;
    text-align: center;
  }
  
  .search-container {
    margin-bottom: 20px;
  }
  
  input {
    padding: 10px;
    width: 300px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  button {
    padding: 10px 20px;
    margin-left: 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #45a049;
  }
  
  .food-list {
    margin-top: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .food-item {
    background-color: #fff;
    padding: 15px;
    margin: 10px;
    width: 80%;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  .food-item h3 {
    margin: 0;
    font-size: 1.2em;
  }
  
  .food-item p {
    color: #555;
  }
  </style>
  