<template>
    <div class="custom-select" @click="toggleDropdown">
      <div class="selected-option">
        <span>{{ displayText }}</span>
      </div>
      <ul v-if="showDropdown" class="dropdown-list">
        <li class="select-all" @click.stop="toggleSelectAll">
          Все статусы
        </li>
        <li v-for="option in options" :key="option" @click.stop="toggleOption(option)">
          <p class="p-lvl-edu">{{ option }}</p>
        </li>
      </ul>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, watch } from 'vue';
  
  const options = ref(['Открыта', 'Закрыта']);
  const selectedOptions = ref(JSON.parse(localStorage.getItem('selectedOptionsCustomSelect')) || []);
  const showDropdown = ref(false);
  
  const toggleDropdown = () => {
    showDropdown.value = !showDropdown.value;
  };
  
  const toggleOption = (option) => {
    if (selectedOptions.value.includes(option)) {
      selectedOptions.value = selectedOptions.value.filter(item => item !== option);
    } else {
      selectedOptions.value.push(option);
    }
    saveToLocalStorage();
  };
  
  const isAllSelected = computed(() => selectedOptions.value.length === options.value.length);
  
  const toggleSelectAll = () => {
    if (isAllSelected.value) {
      selectedOptions.value = [];
    } else {
      selectedOptions.value = [...options.value];
    }
    saveToLocalStorage();
  };
  
  const displayText = computed(() => {
    if (selectedOptions.value.length === 0) return "Все статусы";
    return selectedOptions.value.length === options.value.length ? "Все статусы" : selectedOptions.value.join(", ");
  });
  
  
  const saveToLocalStorage = () => {
    localStorage.setItem('selectedOptionsCustomSelect', JSON.stringify(selectedOptions.value));
  };
  
  watch(selectedOptions, saveToLocalStorage);
  </script>
  
  <style scoped lang="scss">
  @use '@/assets/scss/_fonts.scss' as *;

  @media (max-width: 1624px) {
  
  .custom-select {
  width: 300px !important;
  height: 56px;
  border: 1px solid #d3d3de;
  background-color: white;
  cursor: pointer;
  position: relative;
  border-radius: 8px;
  display: flex;
  align-items: center;
}
}

  @media (max-width:1024px){
    .selected-option {
    width: 100% !important;
    margin: 15px;
    font-family: Gotham Pro;
    font-weight: 400;
    font-size: 16px;
    line-height: 20.8px;
    letter-spacing: 0px;
  }
  .custom-select {
    width: 100% !important;
    height: 56px;
    border: 1px solid #d3d3de;
    background-color: white;
    cursor: pointer;
    position: relative;
    border-radius: 8px;
    display: flex;
    align-items: center;
  }
}
  
  
  .p-lvl-edu {
    border: 1px solid #d3d3de;
    border-radius: 9px;
    padding: 4px;
    &:hover {
      background: #f0f0f0;
    }
  }
  
  .select-all {
    &:hover {
      background: #f0f0f0;
    }
  }
  
  .custom-select {
    width: 517px;
    height: 56px;
    border: 1px solid #d3d3de;
    background-color: white;
    cursor: pointer;
    position: relative;
    border-radius: 8px;
    display: flex;
    align-items: center;
  }
  
  .selected-option {
    width: 100%;
    margin: 15px;
    font-family: Gotham Pro;
    font-weight: 400;
    font-size: 16px;
    line-height: 20.8px;
    letter-spacing: 0px;
  }
  
  .dropdown-list {
    position: absolute;
    top: 100%;
    left: 0;
    width: 100%;
    background: white;
    border: 1px solid #d3d3de;
    border-radius: 5px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    z-index: 10;
    font-family: Gotham Pro;
    font-weight: 400;
    font-size: 16px;
    line-height: 20.8px;
    letter-spacing: 0px;
    display: flex;
  }
  
  li {
    padding: 10px 15px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  </style>
  