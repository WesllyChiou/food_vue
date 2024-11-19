<template>
  <div class="app-container">
    <h1>熱量查詢系統</h1>
    <div class="search-container">
      <input v-model="searchQuery" placeholder="輸入食物名稱或俗名" />
      <button @click="searchFood">搜尋</button>
    </div>

    <!-- 顯示進度條 -->
    <div v-if="isLoading" class="progress-bar">
      <p>搜尋中...</p>
    </div>

    <div v-if="foods.length > 0" class="food-list">
      <div
        v-for="food in foods"
        :key="food._id"
        class="food-item"
        @click="openExerciseModal(food)"
      >
        <h3>{{ food.樣品名稱 }}</h3>
        <p>俗名: {{ food.俗名 }}</p>
        <p>熱量: {{ food['修正熱量(kcal)'] }} 大卡</p>
      </div>
    </div>

    <!-- 查無資料時顯示 -->
    <div v-else>
      <p>查無資料</p>
    </div>

    <!-- 運動建議彈窗 -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close-btn" @click="closeModal">&times;</span>
        <h3>{{ selectedFood.樣品名稱 }} 的運動建議</h3>
        <p>熱量：{{ selectedFood['修正熱量(kcal)'] }} 大卡</p>
        <p>請輸入體重 (kg)：<input v-model="weight" type="number" placeholder="輸入體重" /></p>
        <button @click="calculateBMR">計算BMR</button>
        <p v-if="bmr">您的基礎代謝率 (BMR) 為：{{ bmr }} 大卡</p>
        <p>您需要進行以下運動來消耗這些熱量：</p>
        <ul>
          <li v-for="exercise in exerciseTypes" :key="exercise">
            {{ exercise }}：{{ calculateExerciseTime(selectedFood['修正熱量(kcal)'], exercise) }} 分鐘
          </li>
        </ul>
      </div>
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
      isLoading: false, // 顯示進度條
      showModal: false, // 是否顯示彈窗
      selectedFood: null, // 被選中的食物資料
      exerciseTypes: [
        "跑步",
        "游泳",
        "腳踏車",
        "籃球",
        "瑜珈",
        "重訓",
        "拳擊",
        "柔道",
        "跆拳道",
        "滑板",
        "街舞",
        "直排輪",
        "羽球",
        "桌球",
        "網球",
        "空手道",
      ], // 新增的運動類型
      weight: null, // 用戶輸入的體重
      bmr: null, // 計算出來的基礎代謝率 (BMR)
    };
  },
  methods: {
    async searchFood() {
      if (this.searchQuery.trim()) {
        this.foods = [];
        this.isLoading = true;

        try {
          const response = await axios.get("https://food-server-ycm2.onrender.com/api/search", {
            params: { query: this.searchQuery },
          });
          this.foods = response.data;
        } catch (error) {
          console.error("搜尋錯誤:", error);
        } finally {
          this.isLoading = false;
        }
      }
    },

    // 當使用者點選某筆食物資料時開啟彈窗
    openExerciseModal(food) {
      this.selectedFood = food;
      this.showModal = true;
    },

    // 關閉彈窗
    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.weight = null; // 清空體重欄位
      this.bmr = null; // 清空BMR
    },

    // 根據熱量計算運動時間
    calculateExerciseTime(kcal, exerciseType) {
      // 假設每種運動每分鐘消耗的熱量 (kcal)
      const calorieBurnRate = {
        跑步: 10,
        游泳: 7,
        腳踏車: 5,
        籃球: 6,
        瑜珈: 3,
        重訓: 8,
        拳擊: 12,
        柔道: 10,
        跆拳道: 9,
        滑板: 4,
        街舞: 5,
        直排輪: 7,
        羽球: 6,
        桌球: 4,
        網球: 7,
        空手道: 11,
      };

      // 計算運動時間：熱量 ÷ 每分鐘消耗的熱量
      const burnRate = calorieBurnRate[exerciseType] || 0;
      return burnRate ? Math.round(kcal / burnRate) : 0;
    },

    // 計算基礎代謝率 (BMR)
    calculateBMR() {
      if (this.weight && this.selectedFood) {
        // 假設以哈里斯-貝尼迪克特公式計算 BMR
        // 男性：BMR = 88.362 + (13.397 * 體重) + (4.799 * 身高) - (5.677 * 年齡)
        // 女性：BMR = 447.593 + (9.247 * 體重) + (3.098 * 身高) - (4.330 * 年齡)
        // 這裡假設身高150cm，年齡30歲，並且性別為女性（可以根據需要調整）

        const height = 150; // 假設身高150 cm
        const age = 30; // 假設年齡30歲
        const isMale = false; // 假設是女性

        let bmr = 0;
        if (isMale) {
          bmr = 88.362 + (13.397 * this.weight) + (4.799 * height) - (5.677 * age);
        } else {
          bmr = 447.593 + (9.247 * this.weight) + (3.098 * height) - (4.330 * age);
        }
        this.bmr = Math.round(bmr);
      }
    },
  },
};
</script>

<style scoped>
/* 樣式可以根據需要進行自定義 */
.app-container {
  font-family: Arial, sans-serif;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
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
  cursor: pointer;
}

/* 食物清單項目 */
.food-item {
  background-color: #f9f9f9;
  margin: 10px 0;
  padding: 15px;
  border-radius: 5px;
  cursor: pointer;
}

.food-item p {
  color: #555;
}

/* 進度條的樣式 */
.progress-bar {
  margin-top: 20px;
  width: 100%;
  text-align: center;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 4px;
}

/* 運動建議彈窗 */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  cursor: pointer; /* 點擊外部可關閉彈窗 */
}


/* 防止點擊內容區域關閉視窗 */
.modal-content {
  background-color: white;
  padding: 30px;
  border-radius: 8px;
  width: 500px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  cursor: auto; /* 點擊內容區域不會關閉彈窗 */
  max-height: 80vh; /* 設定最大高度 */
  overflow-y: auto; /* 讓內容區域可滾動 */
}

/* 關閉按鈕 */
.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 30px;
  cursor: pointer;
}
</style>
