<template>
  <div class="app-container">
    <h1>熱量查詢系統</h1>
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

        <!-- 顯示計算BMR的區域，只有在點擊計算BMR後才顯示 -->
        <div v-if="showBMRFields">
          <div>
            <p>請輸入體重 (kg)：<input v-model="weight" type="number" placeholder="輸入體重" @input="updateBMRAndExerciseTime" /></p>
            <p>請輸入身高 (cm)：<input v-model="height" type="number" placeholder="輸入身高" @input="updateBMRAndExerciseTime" /></p>
            <p>請輸入年齡 (歲)：<input v-model="age" type="number" placeholder="輸入年齡" @input="updateBMRAndExerciseTime" /></p>
            <p>選擇性別： 
              <select v-model="gender" @change="updateBMRAndExerciseTime">
                <option value="male">男</option>
                <option value="female">女</option>
              </select>
            </p>
          </div>

          <p v-if="bmr">您的基礎代謝率 (BMR) 為：{{ bmr }} 大卡</p>
        </div>

        <!-- 計算BMR按鈕，移到下方 -->
        <div v-if="!showBMRFields">
          <button @click="toggleBMRFields">計算BMR</button>
        </div>

        <!-- 運動建議清單，顯示在計算BMR區域下方 -->
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
      searchQuery: "",
      foods: [],
      isLoading: false,
      showModal: false,
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
      weight: null,
      height: null,
      age: null,
      gender: null,
      bmr: null,
      showBMRFields: false, // BMR欄位是否顯示
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
      this.showBMRFields = false; // 每次打開彈窗時，隱藏BMR欄位
      this.bmr = null; // 重置BMR顯示
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.weight = null;
      this.height = null;
      this.age = null;
      this.gender = null;
      this.bmr = null;
      this.showBMRFields = false; // 重置BMR輸入欄位顯示
    },

    closeModalOnOutsideClick(event) {
      if (event.target === event.currentTarget) {
        this.closeModal();
      }
    },

    toggleBMRFields() {
      this.showBMRFields = !this.showBMRFields;
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

    updateBMRAndExerciseTime() {
      if (this.weight && this.height && this.age && this.gender) {
        let bmr = 0;
        if (this.gender === 'male') {
          bmr = 88.362 + (13.397 * this.weight) + (4.799 * this.height) - (5.677 * this.age);
        } else {
          bmr = 447.593 + (9.247 * this.weight) + (3.098 * this.height) - (4.330 * this.age);
        }
        this.bmr = Math.round(bmr);
      } else {
        this.bmr = null;
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
}

.food-item {
  padding: 15px;
  margin: 10px 0;
  background-color: #f9f9f9;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.food-item:hover {
  background-color: #f1f1f1;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 100%;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
</style>
