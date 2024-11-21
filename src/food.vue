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
      weight: null, // 體重
      height: null, // 身高
      age: null, // 年齡
      gender: "female", // 預設性別為女性
      activityLevel: 1.2, // 用戶選擇的活動水平
      bmr: null, // 計算出來的基礎代謝率 (BMR)
      tdee: null, // 計算出來的每日總能量消耗 (TDEE)
      showBMRFields: true, // 顯示BMR欄位
      exerciseCaloriesPerKg: {
        "慢走": { rate: 0.035 },
        "快走": { rate: 0.04 },
        "爬樓梯": { rate: 0.06 },
        "跑步": { rate: 0.082 },
        "游泳": { rate: 0.065 },
        "腳踏車": { rate: 0.045 },
        "籃球": { rate: 0.05 },
        "瑜珈": { rate: 0.03 },
        "拳擊": { rate: 0.08 },
        "高爾夫": { rate: 0.02 },
        "羽毛球": { rate: 0.04 },
        "跳繩": { rate: 0.1 }
      },
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
      this.calculateExerciseTimes(food['修正熱量(kcal)']);
      this.showModal = true;
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.resetBMRInputs();
    },

    closeModalOnOutsideClick(event) {
      if (event.target === event.currentTarget) {
        this.closeModal();
      }
    },

    resetBMRInputs() {
      this.weight = null;
      this.height = null;
      this.age = null;
      this.gender = "female";
      this.activityLevel = 1.2;
      this.bmr = null;
      this.tdee = null;
      this.exerciseTimes = {};
    },

    toggleBMRFields() {
      this.showBMRFields = !this.showBMRFields;
    },

    updateBMR() {
      if (this.weight && this.height && this.age) {
        const bmr = this.calculateBMR(this.weight, this.height, this.age, this.gender);
        this.bmr = Math.round(bmr); // 四捨五入為整數
        this.tdee = Math.round(bmr * this.activityLevel); // 四捨五入為整數
      }
    },

    calculateBMR(weight, height, age, gender) {
      if (gender === 'female') {
        return 655 + (9.6 * weight) + (1.8 * height) - (4.7 * age);
      } else {
        return 66 + (13.7 * weight) + (5 * height) - (6.8 * age);
      }
    },

    calculateExerciseTimes(calories) {
  const times = {};
  
  // 遍歷所有運動
  this.exerciseTypes.forEach((exercise) => {
    const caloriesPerKg = this.exerciseCaloriesPerKg[exercise]?.rate || 0;  // 每公斤的消耗熱量
    const caloriesIn30Minutes = caloriesPerKg * this.weight * 0.5;  // 30分鐘消耗的熱量（0.5小時）
    const targetCalories = calories;  // 食物的總熱量
    
    // 計算需要幾次運動來消耗這些食物的熱量，保留小數點第一位
    const neededSessions = targetCalories / caloriesIn30Minutes; // 直接計算，不進行四捨五入
    const sessionsWithOneDecimal = parseFloat(neededSessions.toFixed(1)); // 保留小數點第一位
    
    // 設定每個運動的所需時間為 30 分鐘的次數
    times[exercise] = (sessionsWithOneDecimal * 30).toFixed(1);  // 每次運動時間是 30 分鐘，並保留一位小數
  });

  this.exerciseTimes = times;
}

  },
};
</script>

<style scoped>
/* Your styles here */
</style>


<style scoped>
.app-container {
  font-family: Arial, sans-serif;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  font-size: 24px;
  margin-bottom: 20px;
}

.search-container {
  margin-bottom: 20px;
}

input {
  padding: 10px;
  width: 70%;
  max-width: 400px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
}

button {
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.food-list {
  display: flex;
  flex-direction: column;
  gap: 10px; /* 調整項目間距 */
}

.food-item {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
  text-align: left;
  transition: transform 0.2s, box-shadow 0.2s;
}

.food-item:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.food-item h3 {
  margin: 0;
  font-size: 18px;
}

.food-item p {
  margin: 5px 0;
  color: #555;
  font-size: 14px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  text-align: left;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  max-height: 80vh;  /* 限制模態框的最大高度 */
  overflow-y: auto;  /* 允許垂直滾動 */
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  cursor: pointer;
  color: #333;
}

.exercise-container {
  display: flex;
  flex-wrap: wrap;
}

.exercise-item {
  margin: 5px;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 4px;
  font-size: 16px;
}

input[type="number"],
.styled-select {
  margin: 5px 0;
  padding: 8px;
  width: calc(100% - 16px);
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.styled-select {
  width: 100%;
}

button {
  margin-top: 10px;
}

.progress-bar {
  text-align: center;
  font-size: 16px;
  color: #666;
}

.progress-bar p {
  margin: 0;
}

p {
  font-size: 14px;
  line-height: 1.5;
}

@media (max-width: 768px) {
  input {
    width: 100%;
  }

  button {
    width: 100%;
    padding: 10px;
  }

  .modal-content {
    width: 95%;
  }
}
</style>

