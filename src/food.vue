<template>
  <div class="app-container">
    <h1>熱量查詢系統</h1>
    <div class="search-container">
      <input v-model="searchQuery" placeholder="輸入食物名稱或俗名" />
      <button @click="debouncedSearchFood">搜尋</button>
    </div>

    <!-- 顯示進度條 -->
    <div v-if="isLoading" class="progress-bar">
      <p>搜尋中...</p>
    </div>

    <!-- 顯示食物資料 -->
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
    <div v-else-if="!isLoading && !foods.length && searchPerformed">
      <p>查無資料，請檢查輸入內容。</p>
    </div>

    <!-- 運動建議彈窗 -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close-btn" @click="closeModal">&times;</span>
        <h3>{{ selectedFood.樣品名稱 }} 的運動建議</h3>
        <p>熱量：{{ selectedFood['修正熱量(kcal)'] }} 大卡</p>

        <!-- 運動時間計算 -->
        <p>您需要進行以下運動來消耗這些熱量：</p>
        <ul>
          <li v-for="(time, exercise) in exerciseTimes" :key="exercise">
            {{ exercise }}：{{ time }} 分鐘
          </li>
        </ul>

        <!-- 計算基礎代謝率 -->
        <button @click="toggleBmrFields">計算基礎代謝率</button>
        <div v-if="showBmrFields" class="bmr-form">
          <label>性別：
            <select v-model="userInfo.gender">
              <option value="male">男</option>
              <option value="female">女</option>
            </select>
          </label>
          <label>年齡：<input v-model.number="userInfo.age" type="number" /></label>
          <label>身高 (cm)：<input v-model.number="userInfo.height" type="number" /></label>
          <label>體重 (kg)：<input v-model.number="userInfo.weight" type="number" /></label>
          <label>活動因子：
            <select v-model="userInfo.activityFactor">
              <option value="1.2">很少運動 (久坐)</option>
              <option value="1.375">輕度運動 (運動少於三天)</option>
              <option value="1.55">中等運動 (每週運動三到五天)</option>
              <option value="1.725">重度運動 (每週運動六天)</option>
              <option value="1.9">非常重度運動 (每天運動)</option>
            </select>
          </label>
          <p>基礎代謝率：{{ calculateBMR() }} 大卡/天</p>
          <button @click="updateExerciseTimes">更新運動時間</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import debounce from "lodash/debounce";

export default {
  data() {
    return {
      searchQuery: "",
      foods: [],
      isLoading: false,
      showModal: false,
      showBmrFields: false,
      searchPerformed: false,
      selectedFood: null,
      exerciseTimes: {},
      exerciseCalories: {
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
      },
      userInfo: {
        gender: "male",
        age: 25,
        height: 170,
        weight: 65,
        activityFactor: 1.2,
      },
    };
  },
  created() {
    this.debouncedSearchFood = debounce(this.searchFood, 300);
  },
  methods: {
    async searchFood() {
      if (!this.searchQuery.trim()) {
        alert("請輸入食物名稱或俗名！");
        return;
      }
      this.foods = [];
      this.isLoading = true;
      this.searchPerformed = true;

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
    },

    openExerciseModal(food) {
      this.selectedFood = food;
      this.showModal = true;
      this.calculateExerciseTimes(food["修正熱量(kcal)"]);
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.showBmrFields = false;
    },

    toggleBmrFields() {
      this.showBmrFields = !this.showBmrFields;
    },

    calculateExerciseTimes(kcal) {
      this.exerciseTimes = Object.fromEntries(
        Object.entries(this.exerciseCalories).map(([exercise, rate]) => [
          exercise,
          Math.round(kcal / rate),
        ])
      );
    },

    calculateBMR() {
      const { gender, age, height, weight, activityFactor } = this.userInfo;
      if (!age || !height || !weight) {
        alert("請完整輸入基礎代謝率相關資料！");
        return 0;
      }
      const base = gender === "male"
        ? 88.36 + 13.4 * weight + 4.8 * height - 5.7 * age
        : 447.6 + 9.2 * weight + 3.1 * height - 4.3 * age;
      return Math.round(base * activityFactor);
    },

    updateExerciseTimes() {
      if (!this.selectedFood) return;
      const bmr = this.calculateBMR();
      if (bmr) {
        this.calculateExerciseTimes(this.selectedFood["修正熱量(kcal)"] + bmr);
        alert("運動時間已更新！包含BMR。");
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
  cursor: pointer;
}

.food-item h3 {
  margin: 
  }
</style>