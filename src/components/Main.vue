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
        <!-- Пагинация -->
        <div class="pagination" v-if="pages_count > 1">
          <!-- Кнопка предыдущей страницы -->
          <button 
            :disabled="page <= 1" 
            @click="goToPage(page - 1)" 
            class="pagination-btn">
            <img :src="backArrow" alt="Предыдущая">
          </button>
          
          <span class="page-info">
            Страница {{ page }} из {{ pages_count }}
          </span>
          
          <!-- Кнопка следующей страницы -->
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
import MiniIconDoc from "@/assets/images/Icon.png"
import MiniIconSearch from "@/assets/images/IconSearch.png"
import nextArrow from "@/assets/images/next-arrow.png"
import backArrow from "@/assets/images/back-arrow.png"
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
const pages_count = ref();  // Количество страниц, получаем с API
const totalRecords = ref(); // Всего записей, получаем с API

// Функция для получения данных школ
const fetchSchools = async () => {
  isLoading.value = true;

  try {
    const selectedOptions = JSON.parse(localStorage.getItem('selectedOptions')) || [];
    const selectedOptionsCust = JSON.parse(localStorage.getItem('selectedOptionsCustomSelect')) || [];

    const response = await axios.get(`https://schooldb.skillline.ru/api/schools`, {
      params: {
        count: count.value,
        page: page.value,
      },
      headers: {
        "accept": "application/json",
        "X-CSRF-TOKEN": ""
      }
    });

    if (response.status === 200) {
      const filteredData = filterSchools(response.data, selectedOptions, selectedOptionsCust);
      schools.value = filteredData;
      pages_count.value = schools.value.data.pages_count;
      totalRecords.value = schools.value.data.total_count;
      page.value = schools.value.data.page;
    } else {
      handleError(response.status);
    }
  } catch (error) {
    handleError(error.response?.status || 500);
  } finally {
    isLoading.value = false;
  }
};

const filteredSchools = computed(() => {
  if (!searchQuery.value.trim()) return schools.value.data?.list || []; // Если поиск пустой, показываем все

  return (schools.value.data?.list || []).filter(school => {
    const name = school.edu_org.short_name || ""; // Если `short_name` отсутствует, подставляем пустую строку
    const region = school.edu_org.region?.name || ""; // Проверяем `region.name`
    const address = school.edu_org.contact_info?.post_address || ""; // Проверяем `contact_info.post_address`

    const query = searchQuery.value.toLowerCase();
    return (
      name.toLowerCase().includes(query) ||
      region.toLowerCase().includes(query) ||
      address.toLowerCase().includes(query)
    );
  });
});


const handleSearch = () => {
  page.value = 1; // Сбрасываем страницу на первую
  localStorage.setItem('searchQuery', searchQuery.value); // Сохраняем запрос в localStorage
  fetchSchools(); // Перезапрашиваем данные после поиска
};


// Функция для проверки, открыта ли школа
const checkSchoolStatus = (endDate) => {
  const currentDate = new Date();
  if (!endDate) {
    return true; // Если end_date не указана, школа считается открытой
  }
  return new Date(endDate) > currentDate; // Если дата больше текущей, школа открыта
};

// Функция фильтрации данных
const filterSchools = (data, selectedOptions, selectedOptionsCust) => {
  if (!selectedOptions && !selectedOptionsCust || (selectedOptions.length === 0 && selectedOptionsCust.length === 0)) {
    return data; // Если фильтры не заданы, возвращаем все данные
  }

  const filteredList = data.data.list.filter((school) => {
    // Фильтрация по уровню образования
    const eduLevel = school.supplements
      ?.flatMap((supplement) => supplement.educational_programs)
      ?.find((program) => program?.edu_level?.short_name)
      ?.edu_level?.short_name;

    const levelMatch = selectedOptions.includes(eduLevel);

    // Фильтрация по статусу (открыта/закрыта)
    const isOpen = checkSchoolStatus(school.end_date);
    const statusMatch = selectedOptionsCust.includes('Открыта') && isOpen || selectedOptionsCust.includes('Закрыта') && !isOpen;

    // Возвращаем школу, если оба условия (уровень образования и статус) совпадают
    return (selectedOptions.length === 0 || levelMatch) && (selectedOptionsCust.length === 0 || statusMatch);
  });

  return {
    ...data,
    data: {
      ...data.data,
      list: filteredList,
    },
  };
};

const handleError = (status) => {
  switch (status) {
    case 400:
      errorMessage.value = "Ошибка валидации";
      break;
    case 404:
      errorMessage.value = "Страница не найдена";
      break;
    case 429:
      errorMessage.value = "Слишком частые запросы";
      break;
    case 500:
      errorMessage.value = "Ошибка сервера";
      break;
    default:
      errorMessage.value = `Неизвестная ошибка (${status}).`;
  }
};

// Функции обновления количества школ на странице и другие
const updateSchoolsCount = () => {
  page.value = 1;
  fetchSchools();
};

const isAnySelected = computed(() => selectedSchools.value.length > 0);

const toggleSelectAll = () => {
  if (isAnySelected.value) {
    selectedSchools.value = [];
  } else {
    selectedSchools.value = schools.value.data.list.map((_, index) => index);
  }
};

const toggleSingleSelection = (index) => {
  if (selectedSchools.value.includes(index)) {
    selectedSchools.value = selectedSchools.value.filter((i) => i !== index);
  } else {
    selectedSchools.value.push(index);
  }
};

const findEduLevelCode = (supplements) => {
  if (!Array.isArray(supplements)) return "—";
  for (const supplement of supplements) {
    if (Array.isArray(supplement.educational_programs)) {
      const program = supplement.educational_programs.find(p => p?.edu_level?.short_name);
      if (program) return program.edu_level.short_name;
    }
  }
  return "—";
};

const goToPage = (newPage) => {
  if (newPage >= 1 && newPage <= pages_count.value && pages_count.value) {
    page.value = newPage;
    fetchSchools();
  }
};

onMounted(() => {
  fetchSchools();
});
</script>
