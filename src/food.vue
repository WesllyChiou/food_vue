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
        <p>您需要進行以下運動來消耗這些熱量：</p>
        <ul>
          <li v-for="exercise in exerciseTypes" :key="exercise">
            {{ exercise }}：{{ calculateExerciseTime(selectedFood['修正熱量(kcal)'], exercise) }} 分鐘
          </li>
        </ul>
        <button @click="toggleBmrFields">計算基礎代謝率</button>

        <!-- 基礎代謝率欄位 -->
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
          <p>基礎代謝率：{{ calculateBMR() }} 大卡/天</p>
          <button @click="updateExerciseTimes">更新運動時間</button>
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
      showBmrFields: false, // 控制基礎代謝率欄位顯示
      selectedFood: null,
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
      ],
      userInfo: {
        gender: "male",
        age: 25,
        height: 170,
        weight: 65,
      },
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
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.showBmrFields = false;
    },

    toggleBmrFields() {
      this.showBmrFields = !this.showBmrFields;
    },

    calculateExerciseTime(kcal, exerciseType) {
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

      const burnRate = calorieBurnRate[exerciseType] || 0;
      return burnRate ? Math.round(kcal / burnRate) : 0;
    },

    calculateBMR() {
      const { gender, age, height, weight } = this.userInfo;
      if (gender === "male") {
        return Math.round(88.36 + 13.4 * weight + 4.8 * height - 5.7 * age);
      } else if (gender === "female") {
        return Math.round(447.6 + 9.2 * weight + 3.1 * height - 4.3 * age);
      }
      return 0;
    },

    updateExerciseTimes() {
      const bmr = this.calculateBMR();
      if (bmr > 0) {
        this.exerciseTypes = this.exerciseTypes.map((exercise) => {
          const newTime = this.calculateExerciseTime(this.selectedFood['修正熱量(kcal)'] + bmr, exercise);
          return `${exercise}：${newTime} 分鐘 (包含BMR)`;
        });
        alert("運動時間已更新！");
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
  margin: 0;
  font-size: 1.2em;
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

/* 彈窗樣式 */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  width: 60%;
  max-width: 600px;
  text-align: left;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  cursor: pointer;
}
</style>
