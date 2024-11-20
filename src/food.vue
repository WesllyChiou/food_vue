<template>
  <div class="app-container">
    <div class="search-container">
      <input v-model="searchQuery" placeholder="搜尋食物..." />
      <button @click="searchFood">搜尋</button>
    </div>

    <div class="food-list">
      <div v-for="food in filteredFoods" :key="food.id" class="food-item" @click="onFoodItemSelected">
        <h3>{{ food.name }}</h3>
        <p>卡路里: {{ food.calories }} kcal</p>
      </div>
    </div>

    <div class="exercise-container">
      <div v-for="exercise in exerciseTypes" :key="exercise" class="exercise-item">
        <h4>{{ exercise }}</h4>
        <p>運動時間：{{ exerciseTimes[exercise] }} 分鐘</p>
      </div>
    </div>

    <!-- BMR 計算區塊 -->
    <div v-if="showBMRFields" class="bmr-container">
      <h2>BMR & TDEE 計算</h2>
      <label>體重(kg):</label>
      <input v-model="weight" type="number" />
      <label>身高(cm):</label>
      <input v-model="height" type="number" />
      <label>年齡:</label>
      <input v-model="age" type="number" />
      <label>性別:</label>
      <select v-model="gender">
        <option value="male">男性</option>
        <option value="female">女性</option>
      </select>
      <label>活動量:</label>
      <select v-model="activityLevel">
        <option value="1.2">極少運動</option>
        <option value="1.375">輕度運動</option>
        <option value="1.55">中度運動</option>
        <option value="1.725">重度運動</option>
        <option value="1.9">極重運動</option>
      </select>
      <button @click="calculateBMR">計算BMR</button>
      <p>BMR: {{ bmr }} kcal</p>
      <p>TDEE: {{ tdee }} kcal</p>
    </div>

    <!-- BMR 按鈕 -->
    <button @click="toggleBMRFields">{{ bmrButtonText }}</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      weight: 60,
      height: '',
      age: '',
      gender: '',
      activityLevel: '1.2',
      showBMRFields: true,  // 控制BMR相關欄位顯示/隱藏
      bmrButtonText: '計算BMR',  // BMR按鈕文字
      bmr: 0,
      tdee: 0,
      exerciseTypes: ['跑步', '游泳', '瑜伽', '健身'],
      exerciseCaloriesPerKg: {
        跑步: { rate: 0.06, caloriesAt60kg: 360 },
        游泳: { rate: 0.08, caloriesAt60kg: 480 },
        瑜伽: { rate: 0.03, caloriesAt60kg: 180 },
        健身: { rate: 0.05, caloriesAt60kg: 300 },
      },
      exerciseTimes: {},
      filteredFoods: [],  // 搜索結果
      foods: [
        { id: 1, name: '蘋果', calories: 52 },
        { id: 2, name: '香蕉', calories: 96 },
        // 更多食物...
      ],
    };
  },
  methods: {
    // 控制顯示/隱藏BMR相關欄位
    toggleBMRFields() {
      if (this.showBMRFields) {
        // 隱藏BMR欄位，顯示BMR按鈕
        this.showBMRFields = false;
        this.bmrButtonText = '顯示BMR欄位';
      } else {
        // 顯示BMR欄位，隱藏BMR按鈕
        this.showBMRFields = true;
        this.bmrButtonText = '計算BMR';
      }
    },

    // 點選食物項目時清空BMR資料
    onFoodItemSelected() {
      this.bmr = 0;
      this.tdee = 0;
      this.weight = 60;  // 重設體重為預設值
      this.clearBMRInputs();
    },

    // 清空BMR計算欄位
    clearBMRInputs() {
      this.height = '';
      this.age = '';
      this.gender = '';
    },

    // 計算BMR
    calculateBMR() {
      if (this.gender === 'male') {
        this.bmr = 10 * this.weight + 6.25 * this.height - 5 * this.age + 5;
      } else {
        this.bmr = 10 * this.weight + 6.25 * this.height - 5 * this.age - 161;
      }
      this.tdee = this.bmr * this.activityLevel;
    },

    // 搜尋食物
    searchFood() {
      this.filteredFoods = this.foods.filter(food =>
        food.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },

    // 根據體重和運動消耗的卡路里計算運動時間
    calculateExerciseTimes(calories) {
      const exerciseTimes = {};

      this.exerciseTypes.forEach((exercise) => {
        const rate = this.exerciseCaloriesPerKg[exercise]?.rate || 0;
        const caloriesAt60kg = this.exerciseCaloriesPerKg[exercise]?.caloriesAt60kg || 0;
        const time = calories / caloriesAt60kg * 60; // 計算需要的時間（分鐘）

        exerciseTimes[exercise] = time.toFixed(1); // 保留1位小數
      });

      this.exerciseTimes = exerciseTimes;
    }
  }
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
.exercise-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px; /* 間距 */
  margin-top: 10px;
}

.exercise-item {
  padding: 8px 12px;
  background-color: #f0f0f0;
  border-radius: 4px;
  font-size: 14px;
  text-align: center;
  border: 1px solid #ccc;
  flex: 0 1 calc(33.33% - 10px); /* 每行最多顯示三個 */
  box-sizing: border-box;
}

</style>

