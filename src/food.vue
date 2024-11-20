<template>
  <div class="app-container">
    <h1>食物熱量查詢</h1>
    <div class="search-container">
      <input v-model="searchQuery" placeholder="輸入食物名稱或俗名" />
      <button @click="searchFood">搜尋</button>
    </div>

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
        <p>水分: {{ food.水分 }}</p>
        <p>蛋白: {{ food['粗蛋白(g)'] }}</p>
        <p>脂肪: {{ food['粗脂肪(g)'] }}</p>
        <p>碳水化合物: {{ food['總碳水化合物(g)'] }}</p>
      </div>
    </div>

    <div v-else>
      <p>查無資料</p>
    </div>

    <!-- 運動建議彈窗 -->
    <div v-if="showModal" class="modal" @click="closeModalOnOutsideClick">
      <div class="modal-content" @click.stop>
        <span class="close-btn" @click="closeModal">&times;</span>
        <h3>{{ selectedFood.樣品名稱 }} 的運動建議</h3>
        <p>熱量：{{ selectedFood['修正熱量(kcal)'] }} 大卡</p>

        <!-- 顯示計算BMR和TDEE的區域 -->
        <div v-if="showBMRFields">
          <div>
            <p>請輸入體重 (kg)：<input v-model="weight" type="number" placeholder="輸入體重" @input="updateBMR" /></p>
            <p>請輸入身高 (cm)：<input v-model="height" type="number" placeholder="輸入身高" @input="updateBMR" /></p>
            <p>請輸入年齡 (歲)：<input v-model="age" type="number" placeholder="輸入年齡" @input="updateBMR" /></p>
            <p>選擇性別： 
              <select v-model="gender" @change="updateBMR" class="styled-select">
                <option value="male">男</option>
                <option value="female">女</option>
              </select>
            </p>
            <p>選擇活動水平：
              <select v-model="activityLevel" @change="updateBMR" class="styled-select">
                <option value="1.2">久坐少動</option>
                <option value="1.375">輕度活動</option>
                <option value="1.55">中度活動</option>
                <option value="1.725">高度活動</option>
                <option value="1.9">非常活躍</option>
              </select>
            </p>
          </div>

          <p v-if="bmr">您的基礎代謝率 (BMR) 為：{{ bmr }} 大卡</p>
          <p v-if="tdee">您的每日總能量消耗 (TDEE) 為：{{ tdee }} 大卡</p>
        </div>

        <button @click="toggleBMRFields" v-if="!showBMRFields">計算BMR</button>

        <!-- 水平排列的運動建議清單 -->
        <p>您需要進行以下運動來消耗這些熱量：</p>
        <div class="exercise-container">
          <span v-for="(time, exercise) in exerciseTimes" :key="exercise" class="exercise-item">
            {{ exercise }}：{{ time }} 分鐘
          </span>
        </div>
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
        "慢走", "快走", "爬樓梯", "跑步", "游泳", "腳踏車", "籃球", "瑜珈", "拳擊", "高爾夫", "羽毛球", "跳繩"
      ], // 新增的運動類型
      weight: 60, // 預設體重為60公斤
      height: null, // 用戶輸入的身高
      age: null, // 用戶輸入的年齡
      gender: null, // 用戶選擇的性別
      activityLevel: 1.2, // 用戶選擇的活動水平
      bmr: null, // 計算出來的基礎代謝率 (BMR)
      tdee: null, // 計算出來的每日總能量消耗 (TDEE)
      showBMRFields: false, // 是否顯示BMR欄位
      exerciseCaloriesPerKg: {
        "慢走": 1.75,
        "快走": 2.2,
        "爬樓梯": 3.0,
        "跑步": 4.1,
        "游泳": 3.2,
        "腳踏車": 2.0,
        "籃球": 3.0,
        "瑜珈": 1.5,
        "拳擊": 5.7,
        "高爾夫": 1.5,
        "羽毛球": 2.5,
        "跳繩": 5.0
      }, // 每公斤體重的卡路里消耗
      exerciseTimes: {}, // 存儲每種運動消耗熱量所需的時間
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

    openExerciseModal(food) {
      this.selectedFood = food;
      this.showModal = true;
      this.showBMRFields = false;
      this.bmr = null;
      this.tdee = null;

      // 計算每項運動需要多久時間
      this.calculateExerciseTimes(food['修正熱量(kcal)']);
    },

    calculateExerciseTimes(foodCalories) {
      const weight = this.weight || 60; // 使用者體重 (預設60公斤)
      this.exerciseTimes = {};
      
      // 根據每項運動計算時間
      for (const exercise in this.exerciseCaloriesPerKg) {
        const caloriesPerKg = this.exerciseCaloriesPerKg[exercise];
        const time = (foodCalories / (caloriesPerKg * weight)).toFixed(2);
        this.exerciseTimes[exercise] = time;
      }
    },

    // 計算BMR和TDEE的邏輯
    updateBMR() {
      if (this.weight && this.height && this.age && this.gender) {
        let bmr;
        if (this.gender === "male") {
          bmr = 66.5 + (13.75 * this.weight) + (5.003 * this.height) - (6.755 * this.age);
        } else {
          bmr = 655 + (9.563 * this.weight) + (1.850 * this.height) - (4.676 * this.age);
        }

        this.bmr = bmr.toFixed(2);
        this.tdee = (bmr * this.activityLevel).toFixed(2);
      }
    },

    toggleBMRFields() {
      this.showBMRFields = !this.showBMRFields;
    },

    closeModal() {
      this.showModal = false;
    },

    closeModalOnOutsideClick(event) {
      if (event.target === event.currentTarget) {
        this.closeModal();
      }
    },
  },
};
</script>

<style scoped>
/* 樣式部分保持不變 */
</style>
