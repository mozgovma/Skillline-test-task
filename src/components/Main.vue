<template>
  <div class="content">
    <div class="section">
      <div class="section-header">
        <div class="section-title">
          <h3>Таблица учреждений</h3>
        </div>
        <div class="download-btn_search-bar">
          <div class="search-bar">
            <form id="form" @submit.prevent="handleSearch">
              <div class="input-wrapper">
                <input v-model="searchQuery" type="search" id="query" name="q" placeholder="Поиск">
              </div>
            </form>
          </div>
          <div class="download-btn">
            <button class="download-btn-button">
              <img :src="MiniIconDoc" alt="">
              Скачать
            </button>
          </div>
        </div>
      </div>
      <div class="filter">
        <div class="data-input">
            <CustomInput />
        </div>
        <div class="all-types-input">
            <CustomSelect /> 
        </div>
        <div class="all-status-input">
            <CustomSelectForStatus />
        </div>
      </div>

      <div v-if="isLoading" class="loader-container">
        <div class="loader"></div>
      </div>

      <div v-else class="table">
        <table class="custom-table">
          <thead class="table-header">
            <tr class="table-row">
              <th style="width: 25%;" class="table-header-cell">
                <input 
                style="margin: 10px;" 
                type="checkbox"
                :checked="isAnySelected"
                @change="toggleSelectAll"
                :class="{ 'checked-header': isAllSelected }"
                >
                Регион
              </th>
              <th style="width: 25%;" class="table-header-cell">Название</th>
              <th style="width: 25%;" class="table-header-cell">Адрес</th>
              <th style="width: 20%;" class="table-header-cell">Уровень образования</th>
            </tr>
          </thead>
          <tbody class="table-body">
            <tr
            v-if="filteredSchools.length"
            class="table-row" 
            v-for="(school, index) in filteredSchools" 
            :key="index">
              <td class="table-cell">
                <input 
                style="margin: 10px;" 
                type="checkbox"
                :checked="selectedSchools.includes(index)"
                @change="toggleSingleSelection(index)"
                >
                {{ school.edu_org.region?.name || "—" }}
              </td>
              <td class="table-cell">{{ school.edu_org.short_name || "—" }}</td>
              <td class="table-cell">{{ school.edu_org.contact_info?.post_address || "—" }}</td>
              <td class="table-cell">{{ findEduLevelCode(school.supplements) || "—" }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="pagination-footer">

        <div class="pagination" v-if="pages_count > 1">

          <button 
            :disabled="page <= 1" 
            @click="goToPage(page - 1)" 
            class="pagination-btn">
            <img :src="backArrow" alt="Предыдущая">
          </button>
          
          <span class="page-info">
            Страница {{ page }} из {{ pages_count }}
          </span>
          

          <button 
            :disabled="page >= pages_count" 
            @click="goToPage(page + 1)" 
            class="pagination-btn">
            <img :src="nextArrow" alt="Следующая">
          </button>
        </div>

        <div class="schools-per-page">
          <label class="label-for-schoolsCount" for="schoolsCount">Школ на странице: </label>
          <select id="schoolsCount" v-model="count" @change="updateSchoolsCount">
            <option value="10">10</option>
            <option value="20">20</option>
            <option value="30">30</option>
          </select>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import MiniIconDoc from "@/assets/images/Icon.png";
import MiniIconSearch from "@/assets/images/IconSearch.png";
import nextArrow from "@/assets/images/next-arrow.png";
import backArrow from "@/assets/images/back-arrow.png";
import CustomInput from "./CustomInput.vue";
import CustomSelect from "./CustomSelect.vue";
import CustomSelectForStatus from "./CustomSelectForStatus.vue";
import { ref, onMounted, computed } from "vue";
import axios from "axios";

const searchQuery = ref(localStorage.getItem('searchQuery') || "");
const schools = ref([]);
const errorMessage = ref("");
const isLoading = ref(true);
const count = ref(10);
const page = ref(1);
const selectedSchools = ref([]);
const pages_count = ref(0);
const totalRecords = ref(0);

const fetchSchools = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  try {
    const selectedOptions = JSON.parse(localStorage.getItem('selectedOptions')) || [];
    const selectedOptionsCust = JSON.parse(localStorage.getItem('selectedOptionsCustomSelect')) || [];

    const response = await axios.get(`https://schooldb.skillline.ru/api/schools`, {
      params: { count: count.value, page: page.value },
      headers: { "accept": "application/json", "X-CSRF-TOKEN": "" }
    });

    if (response.status === 200) {
      schools.value = filterSchools(response.data, selectedOptions, selectedOptionsCust);
      pages_count.value = schools.value.data.pages_count;
      totalRecords.value = schools.value.data.total_count;
    } else {
      handleError(response.status);
    }
  } catch (error) {
    handleError(error.response?.status || 500);
  } finally {
    isLoading.value = false;
  }
};


const filterSchools = (data, selectedOptions, selectedOptionsCust) => {
  if (!selectedOptions.length && !selectedOptionsCust.length) return data;

  const filteredList = data.data.list.filter(school => {
    const eduLevel = findEduLevelCode(school.supplements);
    const isOpen = checkSchoolStatus(school.end_date);

    const levelMatch = selectedOptions.length ? selectedOptions.includes(eduLevel) : true;
    const statusMatch = selectedOptionsCust.length ? (
      (selectedOptionsCust.includes('Открыта') && isOpen) ||
      (selectedOptionsCust.includes('Закрыта') && !isOpen)
    ) : true;

    return levelMatch && statusMatch;
  });

  return { ...data, data: { ...data.data, list: filteredList } };
};


const checkSchoolStatus = (endDate) => {
  return !endDate || new Date(endDate) > new Date();
};


const findEduLevelCode = (supplements) => {
  if (!Array.isArray(supplements)) return "—";
  for (const supplement of supplements) {
    const program = supplement.educational_programs?.find(p => p?.edu_level?.short_name);
    if (program) return program.edu_level.short_name;
  }
  return "—";
};


const handleError = (status) => {
  const errors = {
    400: "Ошибка валидации",
    404: "Страница не найдена",
    429: "Слишком частые запросы",
    500: "Ошибка сервера",
  };
  errorMessage.value = errors[status] || `Неизвестная ошибка (${status}).`;
};


const filteredSchools = computed(() => {
  if (!searchQuery.value.trim()) return schools.value.data?.list || [];

  const query = searchQuery.value.toLowerCase();
  return schools.value.data?.list.filter(school => {
    const name = school.edu_org.short_name || "";
    const region = school.edu_org.region?.name || "";
    const address = school.edu_org.contact_info?.post_address || "";

    return (
      name.toLowerCase().includes(query) ||
      region.toLowerCase().includes(query) ||
      address.toLowerCase().includes(query)
    );
  });
});


const handleSearch = () => {
  page.value = 1;
  localStorage.setItem('searchQuery', searchQuery.value);
  fetchSchools();
};


const updateSchoolsCount = () => {
  page.value = 1;
  fetchSchools();
};


const toggleSelectAll = () => {
  selectedSchools.value = isAnySelected.value ? [] : schools.value.data.list.map((_, index) => index);
};


const toggleSingleSelection = (index) => {
  selectedSchools.value = selectedSchools.value.includes(index)
    ? selectedSchools.value.filter(i => i !== index)
    : [...selectedSchools.value, index];
};


const isAnySelected = computed(() => selectedSchools.value.length > 0);


const goToPage = (newPage) => {
  if (newPage >= 1 && newPage <= pages_count.value) {
    page.value = newPage;
    fetchSchools();
  }
};


onMounted(fetchSchools);
</script>
