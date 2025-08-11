<script setup>
import { ref, computed } from 'vue'

// 響應式數據
const weight = ref(70)
const height = ref(170)
const unit = ref('metric') // metric 或 imperial

// BMI 分類標準
const bmiCategories = {
  underweight: { min: 0, max: 18.5, label: '體重過輕', color: '#3498db', description: '您的體重低於正常範圍' },
  normal: { min: 18.5, max: 24.9, label: '正常體重', color: '#27ae60', description: '您的體重在健康範圍內' },
  overweight: { min: 25, max: 29.9, label: '體重過重', color: '#f39c12', description: '您的體重略高於正常範圍' },
  obese1: { min: 30, max: 34.9, label: '肥胖 (第一級)', color: '#e67e22', description: '您的體重明顯高於正常範圍' },
  obese2: { min: 35, max: 39.9, label: '肥胖 (第二級)', color: '#e74c3c', description: '您的體重嚴重高於正常範圍' },
  obese3: { min: 40, max: 999, label: '肥胖 (第三級)', color: '#c0392b', description: '您的體重極度高於正常範圍' }
}

// 計算 BMI
const bmi = computed(() => {
  if (unit.value === 'metric') {
    // 公制: kg / (m^2)
    const heightInMeters = height.value / 100
    return weight.value / (heightInMeters * heightInMeters)
  } else {
    // 英制: (lb * 703) / (in^2)
    return (weight.value * 703) / (height.value * height.value)
  }
})

// 獲取 BMI 分類
const bmiCategory = computed(() => {
  const bmiValue = bmi.value
  
  for (const [key, category] of Object.entries(bmiCategories)) {
    if (bmiValue >= category.min && bmiValue < category.max) {
      return { ...category, key }
    }
  }
  
  // 處理極端情況
  if (bmiValue >= 40) {
    return { ...bmiCategories.obese3, key: 'obese3' }
  }
  
  return { ...bmiCategories.underweight, key: 'underweight' }
})

// 健康體重範圍
const healthyWeightRange = computed(() => {
  if (unit.value === 'metric') {
    const heightInMeters = height.value / 100
    const minWeight = 18.5 * (heightInMeters * heightInMeters)
    const maxWeight = 24.9 * (heightInMeters * heightInMeters)
    return {
      min: Math.round(minWeight * 10) / 10,
      max: Math.round(maxWeight * 10) / 10,
      unit: 'kg'
    }
  } else {
    const minWeight = (18.5 * height.value * height.value) / 703
    const maxWeight = (24.9 * height.value * height.value) / 703
    return {
      min: Math.round(minWeight * 10) / 10,
      max: Math.round(maxWeight * 10) / 10,
      unit: 'lb'
    }
  }
})

// 切換單位
const toggleUnit = () => {
  if (unit.value === 'metric') {
    // 轉換到英制
    weight.value = Math.round(weight.value * 2.20462) // kg to lb
    height.value = Math.round(height.value * 0.393701) // cm to in
    unit.value = 'imperial'
  } else {
    // 轉換到公制
    weight.value = Math.round(weight.value / 2.20462) // lb to kg
    height.value = Math.round(height.value / 0.393701) // in to cm
    unit.value = 'metric'
  }
}
</script>

<template>
  <div class="bmi-calculator">
    <header class="header">
      <h1>📊 BMI 計算機</h1>
      <p>計算您的身體質量指數 (Body Mass Index)</p>
    </header>

    <main class="main">
      <div class="calculator-container">
        <div class="input-section">
          <div class="unit-toggle">
            <button 
              :class="{ active: unit === 'metric' }" 
              @click="unit = 'metric'"
              class="unit-btn"
            >
              公制 (kg/cm)
            </button>
            <button 
              :class="{ active: unit === 'imperial' }" 
              @click="unit = 'imperial'"
              class="unit-btn"
            >
              英制 (lb/in)
            </button>
          </div>

          <h2>個人資料</h2>
          
          <div class="input-group">
            <label for="weight">體重 ({{ unit === 'metric' ? 'kg' : 'lb' }})</label>
            <input 
              id="weight"
              type="number" 
              v-model="weight" 
              :min="unit === 'metric' ? 20 : 44"
              :max="unit === 'metric' ? 300 : 660"
              class="input"
              step="0.1"
            >
          </div>

          <div class="input-group">
            <label for="height">身高 ({{ unit === 'metric' ? 'cm' : 'in' }})</label>
            <input 
              id="height"
              type="number" 
              v-model="height" 
              :min="unit === 'metric' ? 100 : 39"
              :max="unit === 'metric' ? 250 : 98"
              class="input"
              step="0.1"
            >
          </div>
        </div>

        <div class="results-section">
          <h2>計算結果</h2>
          
          <div class="bmi-result" :style="{ borderColor: bmiCategory.color }">
            <h3>您的 BMI</h3>
            <div class="bmi-value" :style="{ color: bmiCategory.color }">
              {{ bmi.toFixed(1) }}
            </div>
            <div class="bmi-category" :style="{ color: bmiCategory.color }">
              {{ bmiCategory.label }}
            </div>
            <p class="bmi-description">{{ bmiCategory.description }}</p>
          </div>

          <div class="healthy-weight-section">
            <h3>健康體重範圍</h3>
            <div class="weight-range">
              <span class="range-label">建議體重範圍:</span>
              <span class="range-value">
                {{ healthyWeightRange.min }} - {{ healthyWeightRange.max }} {{ healthyWeightRange.unit }}
              </span>
            </div>
            <p class="range-description">
              基於您的身高，健康體重範圍為 BMI 18.5-24.9
            </p>
          </div>

          <div class="bmi-scale">
            <h3>BMI 分類標準</h3>
            <div class="scale-container">
              <div 
                v-for="(category, key) in bmiCategories" 
                :key="key"
                class="scale-item"
                :style="{ 
                  backgroundColor: category.color,
                  opacity: bmiCategory.key === key ? 1 : 0.7
                }"
              >
                <div class="scale-label">{{ category.label }}</div>
                <div class="scale-range">{{ category.min }}-{{ category.max === 999 ? '∞' : category.max }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="info-section">
        <h3>📋 關於 BMI</h3>
        <div class="info-grid">
          <div class="info-card">
            <h4>什麼是 BMI？</h4>
            <p>身體質量指數 (BMI) 是一個用於評估體重與身高關係的數值，可以幫助判斷體重是否在健康範圍內。</p>
          </div>
          <div class="info-card">
            <h4>計算公式</h4>
            <p><strong>公制:</strong> BMI = 體重(kg) ÷ 身高(m)²</p>
            <p><strong>英制:</strong> BMI = 體重(lb) × 703 ÷ 身高(in)²</p>
          </div>
          <div class="info-card">
            <h4>注意事項</h4>
            <p>BMI 僅供參考，不適用於運動員、孕婦、兒童和老年人。建議諮詢專業醫生以獲得準確的健康評估。</p>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.bmi-calculator {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
}

.header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.header p {
  font-size: 1.2rem;
  opacity: 0.9;
}

.main {
  max-width: 1200px;
  margin: 0 auto;
}

.calculator-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 40px;
  background: white;
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
  margin-bottom: 40px;
}

.unit-toggle {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
}

.unit-btn {
  flex: 1;
  padding: 12px 20px;
  border: 2px solid #e1e5e9;
  background: white;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
}

.unit-btn.active {
  background: #667eea;
  color: white;
  border-color: #667eea;
}

.unit-btn:hover:not(.active) {
  border-color: #667eea;
  background: #f8f9ff;
}

.input-section h2,
.results-section h2 {
  color: #333;
  margin-bottom: 30px;
  font-size: 1.5rem;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

.input-group {
  margin-bottom: 25px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
}

.input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e1e5e9;
  border-radius: 10px;
  font-size: 16px;
  transition: all 0.3s ease;
  background: white;
}

.input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.bmi-result {
  background: #f8f9fa;
  border-radius: 15px;
  padding: 30px;
  margin-bottom: 25px;
  border-left: 5px solid;
  text-align: center;
}

.bmi-result h3 {
  margin-bottom: 15px;
  font-size: 1.2rem;
  color: #333;
}

.bmi-value {
  font-size: 3.5rem;
  font-weight: bold;
  margin-bottom: 10px;
}

.bmi-category {
  font-size: 1.3rem;
  font-weight: 600;
  margin-bottom: 15px;
}

.bmi-description {
  font-size: 0.95rem;
  color: #666;
  line-height: 1.4;
}

.healthy-weight-section {
  background: #f8f9fa;
  border-radius: 15px;
  padding: 25px;
  margin-bottom: 25px;
}

.healthy-weight-section h3 {
  margin-bottom: 15px;
  color: #333;
  font-size: 1.2rem;
}

.weight-range {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  padding: 15px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.range-label {
  font-weight: 500;
  color: #555;
}

.range-value {
  font-weight: bold;
  color: #667eea;
  font-size: 1.1rem;
}

.range-description {
  font-size: 0.9rem;
  color: #666;
  line-height: 1.4;
}

.bmi-scale {
  background: #f8f9fa;
  border-radius: 15px;
  padding: 25px;
}

.bmi-scale h3 {
  margin-bottom: 20px;
  color: #333;
  font-size: 1.2rem;
}

.scale-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 10px;
}

.scale-item {
  padding: 15px;
  border-radius: 10px;
  color: white;
  text-align: center;
  transition: all 0.3s ease;
}

.scale-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.scale-label {
  font-weight: 600;
  margin-bottom: 5px;
  font-size: 0.9rem;
}

.scale-range {
  font-size: 0.8rem;
  opacity: 0.9;
}

.info-section {
  background: white;
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
}

.info-section h3 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
  font-size: 1.5rem;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 25px;
}

.info-card {
  padding: 25px;
  background: #f8f9fa;
  border-radius: 15px;
  border-left: 4px solid #667eea;
}

.info-card h4 {
  color: #333;
  margin-bottom: 15px;
  font-size: 1.1rem;
}

.info-card p {
  color: #666;
  line-height: 1.6;
  margin-bottom: 10px;
}

.info-card p:last-child {
  margin-bottom: 0;
}

@media (max-width: 768px) {
  .calculator-container {
    grid-template-columns: 1fr;
    gap: 30px;
    padding: 20px;
  }
  
  .header h1 {
    font-size: 2rem;
  }
  
  .scale-container {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .info-grid {
    grid-template-columns: 1fr;
  }
  
  .weight-range {
    flex-direction: column;
    gap: 10px;
    text-align: center;
  }
}
</style>
