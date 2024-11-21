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
        <div>
          <div>
            <p>請輸入體重 (kg)：<input v-model.number="weight" type="number" placeholder="輸入體重" /></p>
            <p>請輸入身高 (cm)：<input v-model.number="height" type="number" placeholder="輸入身高" /></p>
            <p>請輸入年齡 (歲)：<input v-model.number="age" type="number" placeholder="輸入年齡" /></p>
            <p>選擇性別：
              <select v-model="gender" class="styled-select">
                <option value="male">男</option>
                <option value="female">女</option>
              </select>
            </p>
            <p>選擇活動水平：
              <select v-model="activityLevel" class="styled-select">
                <option value="1.2">久坐少動</option>
                <option value="1.375">輕度活動</option>
                <option value="1.55">中度活動</option>
                <option value="1.725">高度活動</option>
                <option value="1.9">非常活躍</option>
              </select>
            </p>
          </div>
          <button @click="updateBMR">計算 BMR</button>
          <p v-if="bmr">您的基礎代謝率 (BMR) 為：{{ Math.round(bmr) }} 大卡</p>
          <p v-if="tdee">您的每日總能量消耗 (TDEE) 為：{{ Math.round(tdee) }} 大卡</p>
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
      searchQuery: "",
      foods: [],
      isLoading: false,
      showModal: false,
      selectedFood: null,
      weight: null,
      height: null,
      age: null,
      gender: "female", // 預設為女性
      activityLevel: 1.2,
      bmr: null,
      tdee: null,
    };
  },
  methods: {
    async searchFood() {
      if (this.searchQuery.trim()) {
        this.foods = [];
        this.isLoading = true;
        try {
          const response = await axios.get(
            "https://food-server-ycm2.onrender.com/api/search",
            { params: { query: this.searchQuery } }
          );
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
    updateBMR() {
      if (this.weight && this.height && this.age) {
        const bmr =
          this.gender === "male"
            ? 88.362 + 13.397 * this.weight + 4.799 * this.height - 5.677 * this.age
            : 447.593 + 9.247 * this.weight + 3.098 * this.height - 4.33 * this.age;
        this.bmr = bmr;
        this.tdee = bmr * this.activityLevel;
      }
    },
  },
};
</script>

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
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  cursor: pointer;
  color: #333;
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

