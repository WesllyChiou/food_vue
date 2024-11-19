<template>
  <div class="main-container">
    <!-- 搜尋框 -->
    <div class="search-box">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="搜尋食物"
        @input="debouncedSearch"
      />
    </div>

    <!-- 搜尋結果清單 -->
    <ul class="result-list">
      <li
        v-for="(item, index) in filteredResults"
        :key="index"
        @click="showDetails(item)"
        class="result-item"
      >
        {{ item.name }}
      </li>
    </ul>

    <!-- 彈跳視窗 -->
    <div v-if="isModalVisible" class="modal-overlay" @click="isModalVisible = false">
      <div class="modal-content" @click.stop>
        <h2>{{ selectedItem.name }}</h2>
        <div class="form-group">
          <label for="weight">體重 (kg):</label>
          <input v-model.number="weight" type="number" id="weight" />
        </div>
        <div class="form-group">
          <label for="gender">性別:</label>
          <select v-model="gender" id="gender" class="gender-select">
            <option value="male">男</option>
            <option value="female">女</option>
          </select>
        </div>
        <div class="result">
          <p>基礎代謝率 (BMR): {{ calculateBMR }} 大卡</p>
          <p>食物熱量: {{ selectedItem.calories }} 大卡</p>
        </div>
        <button @click="isModalVisible = false" class="close-btn">關閉</button>
      </div>
    </div>
  </div>
</template>

<script>
import debounce from "lodash.debounce";

export default {
  data() {
    return {
      searchQuery: "",
      filteredResults: [],
      results: [
        { name: "蘋果", calories: 52 },
        { name: "香蕉", calories: 89 },
        { name: "雞胸肉", calories: 165 },
        // 更多食物項目...
      ],
      isModalVisible: false,
      selectedItem: null,
      weight: 0,
      gender: "male",
    };
  },
  computed: {
    calculateBMR() {
      if (this.gender === "male") {
        return 10 * this.weight + 6.25 * 170 - 5 * 25 + 5; // 假設身高170cm，年齡25歲
      } else {
        return 10 * this.weight + 6.25 * 160 - 5 * 25 - 161; // 假設身高160cm，年齡25歲
      }
    },
  },
  methods: {
    debouncedSearch: debounce(function () {
      this.filteredResults = this.results.filter((item) =>
        item.name.includes(this.searchQuery)
      );
    }, 300),
    showDetails(item) {
      this.selectedItem = item;
      this.isModalVisible = true;
    },
  },
};
</script>

<style scoped>
.main-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.search-box input {
  width: 100%;
  padding: 8px;
  font-size: 16px;
}

.result-list {
  list-style: none;
  padding: 0;
}

.result-item {
  padding: 10px;
  border: 1px solid #ccc;
  margin: 5px 0;
  cursor: pointer;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  min-width: 300px;
  max-width: 400px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

.form-group {
  margin-bottom: 15px;
}

.gender-select {
  width: 100%;
  padding: 5px;
}

.result {
  margin-top: 20px;
}

.close-btn {
  background-color: #f44336;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
}
</style>
