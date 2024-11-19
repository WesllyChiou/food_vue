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

        <!-- 顯示計算BMR和TDEE的區域，移到最下方 -->
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

        <!-- 計算BMR按鈕，移到下方 -->
        <button @click="toggleBMRFields" v-if="!showBMRFields">計算BMR</button>

        <!-- 運動建議清單，顯示在計算BMR區域下方 -->
        <p>您需要進行以下運動來消耗這些熱量：</p>
        <ul>
          <li v-for="exercise in exerciseTypes" :key="exercise">
            {{ exercise }}：{{ exerciseTimes[exercise] }} 分鐘
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
      height: null, // 用戶輸入的身高
      age: null, // 用戶輸入的年齡
      gender: null, // 用戶選擇的性別
      activityLevel: 1.2, // 用戶選擇的活動水平
      bmr: null, // 計算出來的基礎代謝率 (BMR)
      tdee: null, // 計算出來的每日總能量消耗 (TDEE)
      showBMRFields: false, // 是否顯示BMR欄位
      exerciseTimes: {
        "跑步": 30,
        "游泳": 45,
        "腳踏車": 40,
        "籃球": 50,
        "瑜珈": 60,
        "重訓": 45,
        "拳擊": 30,
        "柔道": 40,
        "跆拳道": 45,
        "滑板": 30,
        "街舞": 40,
        "直排輪": 35,
        "羽球": 40,
        "桌球": 25,
        "網球": 40,
        "空手道": 50,
      }, // 固定的運動時間
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
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      this.weight = null;
      this.height = null;
      this.age = null;
      this.gender = null;
      this.activityLevel = 1.2;
      this.bmr = null;
      this.tdee = null;
      this.showBMRFields = false;
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
      if (this.weight && this.height && this.age && this.gender) {
        let bmr = 0;
        if (this.gender === 'male') {
          bmr = 88.362 + (13.397 * this.weight) + (4.799 * this.height) - (5.677 * this.age);
        } else {
          bmr = 447.593 + (9.247 * this.weight) + (3.098 * this.height) - (4.330 * this.age);
        }
        this.bmr = Math.round(bmr);

        // 計算TDEE，根據活動水平
        this.tdee = Math.round(this.bmr * this.activityLevel);
      }
    },
  },
};
</script>

<style scoped>
/* 保持原樣式 */
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
}

.food-item {
  padding: 15px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.food-item:hover {
  background-color: #f9f9f9;
}

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
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  max-width: 600px;
  max-height: 80%;
  overflow-y: auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 20px;
  font-size: 20px;
  cursor: pointer;
}

button {
  margin-top: 20px;
}

/* 針對下拉選單的樣式優化 */
select {
  padding: 10px;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  background-color: #fff;
  cursor: pointer;
  transition: border-color 0.3s ease-in-out;
}

select:hover {
  border-color: #4CAF50;
}

select:focus {
  outline: none;
  border-color: #45a049;
}
</style>

