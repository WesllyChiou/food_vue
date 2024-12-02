<template>
  <div class="app-container">
    <h1>食物熱量及運動消耗時間查詢</h1>
       <div class="search-container">
      
        <!-- 在此處加入 @keydown.enter -->
        <input
        v-model="searchQuery"
        placeholder="輸入食物名稱"
        @keyup.enter="searchFood"
      />
      <button @click="searchFood">搜尋</button>
         <!-- 顯示進度條 -->
    <div v-if="isLoading" class="progress-bar">
    <div class="progress-bar-inner"></div> <!-- 進度條本體 -->
   </div>
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
        <p>熱量: {{ food['熱量(kcal)'] }}kcal  修正熱量: {{ food['修正熱量(kcal)'] }}kcal  蛋白: {{ food['蛋白(g)'] }}g  脂肪: {{ food['脂肪(g)'] }}g  碳水化合物: {{ food['碳水化合物(g)'] }}g  糖: {{ food['糖(g)'] }}g  鈉: {{ food['鈉(mg)'] }}mg</p>
        <h4>資料來源:{{ food['資料來源'] }}</h4>
      </div>
    </div>

    <!-- 顯示加載中的訊息 -->
    <div v-if="isLoading">Loading...</div>

    <!-- 顯示錯誤訊息 -->
    <div v-else-if="error">{{ error }}</div>
   <!-- 查無資料提示 -->
   <div v-if="!isLoading && foods.length === 0 && searchQuery.trim()">
      <p>查無資料</p>
    </div>

    <!-- 運動建議彈窗 -->
    <div v-if="showModal" class="modal" @click="closeModalOnOutsideClick">
      <div class="modal-content" @click.stop>
        <span class="close-btn" @click="closeModal">&times;</span>
        <h2>{{ selectedFood.樣品名稱 }} 的運動建議</h2>
        <p class="food-subtitle">熱量：{{ selectedFood['修正熱量(kcal)'] || selectedFood['熱量(kcal)'] }} 大卡</p>

        <!-- 顯示計算BMR和TDEE的區域 -->
        <div v-if="showBMRFields">
          <div >
            <p class="food-subtitle">請輸入體重 (kg)：<input v-model="weight" type="number" placeholder="輸入體重" @input="updateBMR" /></p>
            <p class="food-subtitle">請輸入身高 (cm)：<input v-model="height" type="number" placeholder="輸入身高" @input="updateBMR" /></p>
            <p class="food-subtitle">請輸入年齡 (歲)：<input v-model="age" type="number" placeholder="輸入年齡" @input="updateBMR" /></p>
            <p class="food-subtitle">選擇性別： 
              <select v-model="gender" @change="updateBMR" class="styled-select">
                <option value="male">男</option>
                <option value="female">女</option>
              </select>
            </p>
            <p class="food-subtitle">選擇活動水平：
              <select v-model="activityLevel" @change="updateBMR" class="styled-select">
                <option value="1.2">久坐少動</option>
                <option value="1.375">輕度活動</option>
                <option value="1.55">中度活動</option>
                <option value="1.725">高度活動</option>
                <option value="1.9">非常活躍</option>
              </select>
            </p>
          </div>

          <p class="food-subtitle" v-if="bmr">您的基礎代謝率 (BMR) 為：{{ bmr }} 大卡</p>
          <p  class="food-subtitle" v-if="tdee">您的每日總能量消耗 (TDEE) 為：{{ tdee }} 大卡</p>
        </div>

        <button @click="toggleBMRFields" v-if="!showBMRFields">計算BMR</button>

        <!-- 水平排列的運動建議清單 -->
        <p class="food-subtitle">您可以進行以下運動來消耗{{ selectedFood.樣品名稱 }}的熱量：</p>
        <div class="exercise-container">
          <span v-for="(time, exercise) in exerciseTimes" :key="exercise" class="exercise-item">
            {{ exercise }}：{{ time }} 分鐘
          </span>
        </div>
      </div>
    </div>

    <footer class="app-footer">
      <p><a href="mailto:demotest3.14.1@gmail.com">聯絡信箱</a></p>
      <p class="social-links">
     
    <a
      :href="`https://social-plugins.line.me/lineit/share?url=${encodeURIComponent(shareUrl)}`"
      target="_blank"
      rel="noopener noreferrer"
      class="social-link"
    >
      分享到LINE
    </a>
    <a
      :href="`https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(shareUrl)}`"
      target="_blank"
      rel="noopener noreferrer"
      class="social-link"
    >
      分享到FB
    </a>

     <!-- 分享到 Thread -->
     <a
        href="#"
        @click.prevent="copyToClipboard"
        class="social-link"
      >
        分享到Thread
      </a>

    <a
        href="#"
        @click.prevent="copyToClipboard"
        class="social-link"
      >
        分享到IG
      </a>
  
</p>
      <p>&copy; 2024 HOWHOWFOOD</p>
    </footer>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      shareUrl: 'https://food-vue.onrender.com', // 替换为实际的分享链接
      searchQuery: "", // 用戶輸入的查詢關鍵字
      foods: [], // 顯示查詢結果
      isLoading: false, // 顯示進度條
      showModal: false, // 是否顯示彈窗
      selectedFood: null, // 被選中的食物資料
      viewCount: 0, // 瀏覽人數
      exerciseTypes: [
        "慢走(4公里/時)", "快走、健走(6.0公里/時)", "下樓梯","上樓梯", "慢跑(8公里/時)","快跑(12公里/時)","快跑(16公里/時)", "游泳(慢)","游泳(較快)","騎腳踏車(一般速度，10公里/時)"
        ,"騎腳踏車(快，20公里/時)","騎腳踏車(很快，30公里/時)", "籃球(半場)","籃球(全場)", "瑜珈", "拳擊", "高爾夫", "羽毛球", "跳繩(慢)","跳繩(快)","拖地"
        ,"搬運重物","跳舞(慢)、元極舞","跳舞(快)、國際標準舞","飛盤","排球","保齡球","太極拳","乒乓球","棒壘球","溜直排輪","有氧舞蹈","網球","足球","健康操","划獨木舟","溜輪鞋","騎馬(小跑)","溜冰刀(16公里/時)","爬岩(35公尺/時)","滑雪(16公里/時)","划船比賽"
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
        "慢走(4公里/時)": { rate: 3.5 },
        "快走、健走(6.0公里/時)": { rate: 5.5 },
        "下樓梯": { rate: 3.2 },
        "上樓梯": { rate: 8.4 },
        "慢跑(8公里/時)": { rate: 8.2 },
        "快跑(12公里/時)": { rate: 12.7 },
        "快跑(16公里/時)": { rate: 16.8 },
        "游泳(慢)": { rate: 6.3 },
        "游泳(較快)": { rate:10.1 },
        "騎腳踏車(一般速度，10公里/時)": { rate: 4 },
        "騎腳踏車(快，20公里/時)": { rate: 8.4 },
        "騎腳踏車(很快，30公里/時)": { rate: 12.6 },
        "籃球(半場)": { rate: 6.3 },
        "籃球(全場)": { rate:8.3 },
        "瑜珈": { rate: 3 },
        "拳擊": { rate: 11.4 },
        "高爾夫": { rate: 3.7 },
        "羽毛球": { rate:5.1 },
        "跳繩(慢)": { rate: 8.4 },
        "跳繩(快)": { rate: 12.6 },
        "拖地": { rate: 3.7 }, 
        "搬運重物": { rate: 8.4 } ,
        "跳舞(慢)、元極舞": { rate: 3.1 } ,
        "跳舞(快)、國際標準舞": { rate: 5.3 },
        "飛盤": { rate: 3.2 },
        "排球": { rate:3.6 },
        "保齡球": { rate: 3.6 },  
        "太極拳": { rate: 4.2 }, 
        "乒乓球": { rate: 4.2 }, 
        "棒壘球": { rate: 4.7 },
        "溜直排輪": { rate: 5.1 },
        "有氧舞蹈": { rate: 6.8 },
        "網球": { rate: 6.6 },
        "足球": { rate: 7.7 },
        "健康操": { rate: 4},
        "划獨木舟": { rate: 3.4},
        "溜輪鞋": { rate: 5.1},
        "騎馬(小跑)": { rate: 5.1},
        "溜冰刀(16公里/時)": { rate: 5.9},
        "爬岩(35公尺/時)": { rate: 7},
        "滑雪(16公里/時)": { rate: 7.2},
        "划船比賽": { rate: 12.4}
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
          this.foods = response.data;// 資料加載後賦值
          this.updateSchemaData(); // 資料加載後更新 JSON-LD //for SEO
        } catch (error) {
          console.error("搜尋錯誤:", error);
        } finally {
          this.isLoading = false;
        }
      }
    },

    //for SEO
    updateSchemaData() {
    // 確保有搜尋結果
    if (this.foods && this.foods.length > 0) {
      // 清空之前的 JSON-LD，以防重複
      const existingScripts = document.querySelectorAll('script[type="application/ld+json"]');
      existingScripts.forEach(script => script.remove());

      // 遍歷每一個食物資料
      this.foods.forEach(food => {
        const schemaData = {
          "@context": "https://schema.org",
          "@type": "Food",
          "name": food.樣品名稱,
          "alternateName": food.俗名,
          "calories": food['熱量(kcal)'],
          "nutrition": {
            "@type": "NutritionInformation",
            "calories": food['熱量(kcal)'],
            "modifiedCalories": food['修正熱量(kcal)'],
            "proteinContent": food['粗蛋白(g)'],
            "fatContent": food['粗脂肪(g)'],
            "carbohydrateContent": food['總碳水化合物(g)']
          },
          "source": food['資料來源']
        };

        // 創建 <script> 標籤並插入到 <head> 中
        const script = document.createElement('script');
        script.type = 'application/ld+json';
        script.innerHTML = JSON.stringify(schemaData);
        document.head.appendChild(script);
      });
    }
  },

    openExerciseModal(food) {
      this.selectedFood = food;
      const calories = this.selectedFood?.['修正熱量(kcal)'] || this.selectedFood?.['熱量(kcal)'];
    if (calories) {
      this.calculateExerciseTimes(calories);
    }
      if (!this.bmr || !this.tdee) {// 打開視窗時即計算 BMR 和 TDEE
      this.updateBMR();
     }
      this.showModal = true;
    },

    closeModal() {
      this.showModal = false;
      this.selectedFood = null;
      //this.resetBMRInputs();
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
  const { weight, height, age, gender, activityLevel } = this;

  if (weight && height && age) {
    this.bmr = Math.round(this.calculateBMR(weight, height, age, gender));
    this.tdee = Math.round(this.bmr * activityLevel);

    const calories = this.selectedFood?.['修正熱量(kcal)'] || this.selectedFood?.['熱量(kcal)'];
    if (calories) {
      this.calculateExerciseTimes(calories);
    }
  }
},


    calculateBMR(weight, height, age, gender) {
      if (gender === 'female') {
        return 655 + (9.6 * weight) + (1.8 * height) - (4.7 * age);
      } else {
        return 66 + (13.7 * weight) + (5 * height) - (6.8 * age);
      }
    },

    copyToClipboard() {
      navigator.clipboard.writeText(this.shareUrl).then(() => {
        alert('連結已複製！請貼到 Instagram 分享。');
      });
    },
    calculateExerciseTimes(calories) {
  const times = {};
  
  // 遍歷所有運動
  this.exerciseTypes.forEach((exercise) => {
    const caloriesPerKg = this.exerciseCaloriesPerKg[exercise]?.rate || 0;  // 每公斤的消耗熱量
    const caloriesInMinutes = caloriesPerKg * this.weight * 1;  // 60分鐘消耗的熱量（1小時）例如 75KG*跑步8.2*1小時
    const targetCalories = calories;  // 食物的總熱量 例如346
    
    // 計算需要幾次運動來消耗這些食物的熱量，保留小數點第一位
    const neededSessions = targetCalories / caloriesInMinutes; // 直接計算，不進行四捨五入
     // 檢查是否為 Infinity
     if (neededSessions === Infinity || isNaN(neededSessions)) {
      times[exercise] = "";
    } else {
      const sessionsWithOneDecimal = parseFloat(neededSessions.toFixed(1)); // 保留小數點第一位
      // 設定每個運動的所需時間為 60 分鐘的次數
      times[exercise] = (sessionsWithOneDecimal * 60).toFixed(1);  // 每次運動時間是 60 分鐘，並保留一位小數
    }
  });

  this.exerciseTimes = times;
}


  }
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
  font-size: 40px;
  margin-bottom: 20px;
}

.search-container {
  position: relative;
  width: 100%;
  padding: 10px;
}
#search-field {
  width: 100%;
  padding: 8px;
  font-size: 16px;
  box-sizing: border-box;
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

/* 基本按鈕樣式 */
button {
  background: linear-gradient(45deg, #FFB74D, #FF7043); /* 橙色與暖紅色漸變背景 */
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.3s ease;
  border-radius: 5px;
}

/* 鼠標懸停時的效果 */
button:hover {
  background: linear-gradient(45deg, #FF7043, #FFB74D); /* 反向漸變效果 */
}

.food-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.food-item {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
  text-align: left;
  transition: transform 0.2s, box-shadow 0.2s;
  display: flex;
  align-items: center;
}

.food-item:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.food-item h3 {
  margin: 0;
  font-size: 20px;
}

.food-item p {
  margin: 5px 0;
  color: #555;
  font-size: 18px;
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
  max-width: 1000px;
  width: 100%;
  text-align: left;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  max-height: 80vh;
  overflow-y: auto;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 30px;
  cursor: pointer;
  color: #333;
}

.exercise-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-top: 20px;
}

.exercise-item {
  flex: 0 0 calc(33.33% - 10px);
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f1f1f1;
  text-align: center;
  font-size: 16px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
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
  position: relative;
  width: 100%;
  height: 5px;
  background-color: #ccc;
  margin-top: 5px;
}

#progress-bar::after {
  content: "";
  display: block;
  height: 100%;
  width: 50%;
  background-color: #0f0;
  transition: width 0.5s ease;
}

.progress-bar-inner {
  position: absolute;
  width: 50%;
  height: 100%;
  background-color: #4caf50;
  animation: progress 2s infinite;
}

@keyframes progress {
  0% { width: 0%; }
  100% { width: 100%; }
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

  .exercise-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
    margin-top: 20px;
  }

  .exercise-item {
    flex: 0 0 calc(50% - 10px);
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #f1f1f1;
    text-align: center;
    font-size: 16px;
    white-space: normal;
    overflow: visible;
    text-overflow: clip;
    min-width: 140px;
  }
}

@media (max-width: 480px) {
  .exercise-item {
    flex: 0 0 calc(100% - 10px);
    min-width: 200px;
  }
}

.footer-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin-top: 20px;
  background-color: #f4f4f4;
  border-top: 1px solid #ccc;
  font-size: 12px;
}

.footer-info span {
  margin-right: 10px;
}

.app-footer {
  background-color: #000;
  color: #fff;
  padding: 10px 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  width: 100%;
  box-sizing: border-box;
}

.app-footer p,
.app-footer a {
  margin: 5px 10px;
  text-align: center;
  text-decoration: none;
  color: #fff;
}

.app-footer a:hover {
  color: #00bcd4;
}

.social-links {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.social-link {
  color: #fff;
  text-decoration: none;
  font-size: 14px;
  padding: 5px 10px;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.social-link:hover {
  background-color: #444;
}

.food-subtitle {
  font-size: 18px;
  color: #333;
}

@media (max-width: 768px) {
  .exercise-item {
    flex: 0 0 calc(50% - 10px);
  }
}

@media (min-width: 769px) {
  .exercise-item {
    flex: 0 0 calc(33.33% - 10px);
  }
}



</style>
