<script setup lang="ts">
import { useRouter, useRuntimeConfig } from "nuxt/app";

import { InputDate } from "~/shared/ui/inputs/input-date/index";
import { Search } from "~/shared/ui/search/index";
import type { SearchTypes } from "~/shared/ui/search/type";
import { Toggle } from "~/shared/ui/toggle/index";
import { Select } from "~/shared/ui/select/index";
import { StatusCard } from "~/shared/ui/status-card/index";
import { Empty } from "~/shared/ui/empty/index";

const router = useRouter()
const baseApi = useRuntimeConfig().public.apiBase
const dateInput = ref<Date[]>([])
const searchTypes = ref<SearchTypes>({
  order_number: {
    title: 'Номер заказа',
    placeholder: 'Введите номер заказа',
  },
  psid: {
    title: 'Номер фотосессии',
    placeholder: 'Введите номер фотосессии',
  },
  client_id: {
    title: 'Клиент ID',
    placeholder: 'Введите клиент ID',
  },
  phone: {
    title: 'Телефон',
    placeholder: 'Введите телефон',
  },
  email: {
    title: 'Email',
    placeholder: 'Введите email',
  },
  payer: {
    title: 'Плательщик, ребенок',
    placeholder: 'Введите плательщика, ребенка',
  }
})
const searchType = ref<string>('psid')
const searchQuery = ref<string>('')
const selectYear = ref<string>()
const selectItems = ref<{ value: number, title: string }[]>([
  {
    value: '2024',
    title: '2024',
  },
  {
    value: '2023',
    title: '2023',
  },
  {
    value: '2022',
    title: '2022',
  },
  {
    value: '2021',
    title: '2021',
  },
  {
    value: '2020',
    title: '2020',
  },
  {
    value: '2019',
    title: '2019',
  },
])
const dataList = ref([])

const dateHandler = (date) => {
  const resArr = []

  if (date && date.length) {
    date.forEach(el => {
      const month = (el.getUTCMonth() + 1).toString().padStart(2, "0")
      const day = el.getDate().toString().padStart(2, "0")
      const year = el.getUTCFullYear()

      resArr.push(`${year}${month}${day}`)
    })
  }

  return resArr
}

const queryParams = ref({
  search_type: searchType.value,
  search_value: searchQuery.value,
  date_start: dateHandler(dateInput.value)[0],
  date_finish: dateHandler(dateInput.value)[1],
  year: selectYear.value
})

watch(searchType, newVal => {
  queryParams.value.search_type = newVal
})

watch(searchQuery, newVal => {
  queryParams.value.search_value = newVal
})

watch(dateInput, newVal => {
  queryParams.value.date_start = dateHandler(newVal)[0]
  queryParams.value.date_finish = dateHandler(newVal)[1]
})

watch(selectYear, newVal => {
  queryParams.value.year = newVal
})

const getData = async (isFirstLoad: boolean) => {
  const params = !isFirstLoad ? setQuery() : ''
  const req = await fetch(`${baseApi}method/orders.getTest${params}`)
  const res = await req.json()

  router.push(params)

  if (res.response.data.orders) {
    dataList.value = res.response.data.orders
  } else {
    dataList.value = []
  }
}

const setQuery = () => {
  const query = []

  Object
    .keys(queryParams.value)
    .forEach(key => {
      if (queryParams.value[key]) query.push(`${key}=${queryParams.value[key]}`)
    })

  return query.length > 1 ? `?${query.join('&')}` : `?${query}`
}

const clearSearch = () => {
  router.push('')

  setTimeout(() => {
    getData(false)
  }, 500)
}

const clearFilters = () => {
  searchQuery.value = ''
  dateInput.value = ''
  selectYear.value = ''

  setTimeout(() => {
    getData(false)
  }, 500)
}

const updateSearchType = (event) => {
  searchType.value = event
}

const updateSearchQuery = (event) => {
  searchQuery.value = event
}

onMounted(() => getData(true))
</script>

<template>
  <main>
    <section class="list">
      <Toggle
        class="width100 outline-button mobile"
      >
        Назад
      </Toggle>
      <div class="filters">
        <div class="filters__item block">
          <InputDate
            v-model="dateInput"
            label="Период"
            range
          />
          <Search
            :search-types="searchTypes"
            :search-type="searchType"
            :search-query="searchQuery"
            select-display-value="icon"
            @update:searchType="updateSearchType"
            @update:searchQuery="updateSearchQuery"
            @submit="getData(false)"
            @clear="clearSearch"
          />
        </div>
        <div class="block filters__item">
          <Select
            v-model="selectYear"
            :items="selectItems"
            label="Год"
            editable
          />
          <div class="filters__item-buttons">
            <Toggle
              class="width100"
              @click="getData(false)"
            >
              Показать
            </Toggle>
            <Toggle
              class="width100 outline-button"
              @click="clearFilters"
            >
              Сбросить
            </Toggle>
          </div>
        </div>
      </div>
      <div v-if="dataList.length" class="data">
        <StatusCard
          v-for="item in dataList"
          :key="item.id"
          class="data__item"
          status="success"
        >
          №: {{ item.id }}
        </StatusCard>
      </div>
      <Empty v-else>
        Заказов еще нет
      </Empty>
    </section>
  </main>
</template>

<style lang="scss">
.list{
  display: flex;
  align-items: flex-start;
  gap: 15px;

  @include active-by("lg") {
    flex-wrap: wrap;
  }

  .filters{
    display: flex;
    flex-direction: column;
    gap: 15px;
    max-width: 400px;
    width: 100%;

    @include active-by("lg") {
      max-width: unset;
    }

    &__item{
      display: flex;
      flex-direction: column;
      gap: 10px;

      &-buttons{
        display: flex;
        align-items: stretch;
        gap: 15px;
        width: 100%;
      }
    }

    .block{
      background: white;
      padding: 16px;
    }
  }

  .data{
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 15px;

    &__item{
      padding: 10px;
    }
  }

  .outline-button{
    background: white;
    border: 1px solid #ddd;
    color: #716aca;

    &.mobile{
      display: none;

      @include active-by("lg") {
        display: block;
      }
    }

    &:hover{
      opacity: 0.7;
    }
  }
}
</style>