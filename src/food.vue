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
        "慢走": { rate: 3.5, caloriesAt60kg: 105 }, // 每60kg消耗105卡路里的慢走
        "快走": { rate: 4.0, caloriesAt60kg: 120 },
        "爬樓梯": { rate: 6.0, caloriesAt60kg: 180 },
        "跑步": { rate: 7.0, caloriesAt60kg: 210 },
        "游泳": { rate: 6.5, caloriesAt60kg: 195 },
        "腳踏車": { rate: 4.5, caloriesAt60kg: 135 },
        "籃球": { rate: 5.0, caloriesAt60kg: 150 },
        "瑜珈": { rate: 3.0, caloriesAt60kg: 90 },
        "拳擊": { rate: 8.0, caloriesAt60kg: 240 },
        "高爾夫": { rate: 2.0, caloriesAt60kg: 60 },
        "羽毛球": { rate: 4.0, caloriesAt60kg: 120 },
        "跳繩": { rate: 10.0, caloriesAt60kg: 300 }
      }, // 每公斤體重的卡路里消耗及60kg對應的消耗值
      exerciseTimes: {}, // 存儲每種運動消耗熱量所需的時間
    };
  },

  methods: {
    async searchFood() {
      if (this.searchQuery.trim()) {
        this.foods = [];
        this.isLoading = true;

        try {
          const response = await axios.get(`http://your-api-url?query=${this.searchQuery}`);
          this.foods = response.data;
        } catch (error) {
          console.error(error);
        } finally {
          this.isLoading = false;
        }
      }
    },

    openExerciseModal(food) {
      this.selectedFood = food;
      this.calculateExerciseTimes(food['修正熱量(kcal)']);
      this.showModal = true;
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
    },

    closeModalOnOutsideClick(event) {
      if (event.target === event.currentTarget) {
        this.closeModal();
      }
    },

    toggleBMRFields() {
      this.showBMRFields = !this.showBMRFields;
    },

    updateBMR() {
      const bmr = this.calculateBMR(this.weight, this.height, this.age, this.gender);
      this.bmr = bmr;
      this.tdee = bmr * this.activityLevel;
    },

    calculateBMR(weight, height, age, gender) {
      // 使用 Harris-Benedict 方程式計算 BMR
      if (gender === "male") {
        return 88.362 + 13.397 * weight + 4.799 * height - 5.677 * age;
      } else {
        return 447.593 + 9.247 * weight + 3.098 * height - 4.330 * age;
      }
    },

    calculateExerciseTimes(calories) {
      for (const exercise of this.exerciseTypes) {
        const caloriesAt60kg = this.exerciseCaloriesPerKg[exercise].caloriesAt60kg;
        const minutes = (calories / caloriesAt60kg) * 30; // 基於60kg，計算消耗熱量所需的時間
        this.exerciseTimes[exercise] = Math.round(minutes);
      }
    },
  },
};
</script>

<style scoped>
  .app-container {
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  .search-container {
    margin-bottom: 20px;
  }
  .food-list {
    margin-top: 20px;
  }
  .food-item {
    background-color: #f4f4f4;
    padding: 10px;
    margin: 10px 0;
    border-radius: 5px;
  }
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    width: 80%;
    max-width: 600px;
  }
  .close-btn {
    position: absolute;
    top: 10px;
    right: 10px;
    cursor: pointer;
    font-size: 20px;
  }
  .exercise-container {
    display: flex;
    flex-wrap: wrap;
  }
  .exercise-item {
    margin: 10px;
    padding: 5px 10px;
    background-color: #4caf50;
    color: white;
    border-radius: 5px;
  }
  .styled-select {
    margin-left: 10px;
    padding: 5px;
    border-radius: 5px;
  }
</style>
