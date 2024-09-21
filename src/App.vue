<script setup>
import {computed, onMounted, ref, watch} from "vue";
import axios from "axios";
import {
  Button as AButton,
  Checkbox as ACheckbox,
  Col as ACol,
  ConfigProvider as AConfigProvider,
  DatePicker as ADatePicker,
  Form as AForm,
  FormItem as AFormItem,
  Input as AInput,
  Row as ARow,
  Select as ASelect,
  Table as ATable
} from "ant-design-vue";
import ruRu from "ant-design-vue/es/locale/ru_RU";
import dayjs from "dayjs";
import ru from "dayjs/locale/ru";
import utc from "dayjs/plugin/utc";

dayjs.locale(ru);
dayjs.extend(utc);

const companyArray = [
  {
    id: 1,
    name: "ARANTA Art Supplies",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTI5ODI5MywiaWQiOiIyOGUxZGRlNS1kMzU5LTQ4MDktYTY3OC04ZTdkNDdmZWIyNzgiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwMTg1MzQsInMiOjEwMjYsInNpZCI6ImRjZTNhNzQ5LWU0ZmQtNDkwMC1iYmYyLWJjMzYyODNkOTk4MCIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.ibhHx9zTX066nuHD6dBoTcf0tK3q0_Tv6vhpHFbk6-qmpgBAKmuP1_NXkCTe1vFqINILROWWj6Qx925YPlSUgg",
    telegramToken: "7397979520:AAFH3aY5u-PO9OOegXDl_5hvv1PLUp_3eQ4",
    chatId: "-4587468459",
  },
  {
    id: 2,
    name: "Sunflowers",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTczMjQ1MSwiaWQiOiIwMTkxZGI3Zi03ZjVlLTcyZTgtYjVkYS02M2JiMjM3MmM4ODUiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwNzg0NjMsInMiOjEwMjYsInNpZCI6IjBmMzY0NjAzLWE4MWMtNDNhZC05MjliLTJhZjMxOWFhZTczYyIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.FWLQfZDGiGOEJBFZHrmo5F3Olw6Ms_9YQyqIIUJO2JCzNjoCRrOJMxhj-j1xiGA-lzMXEya9nO0IcJ2wNqk7ZA",
    telegramToken: "6584534762:AAFtICID-IRygrU-pt_A_N7dA5ypkAwqCO8",
    chatId: "-4585796755",
  },
  {
    id: 3,
    name: "Ne Vi",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTczMjUzMiwiaWQiOiIwMTkxZGI4MC1iYzI3LTcxOTMtYTRhYS1iZGJiZWVhODQ2ZmUiLCJpaWQiOjE5NjI0NzM2LCJvaWQiOjQxMjc0NjcsInMiOjEwMjYsInNpZCI6Ijg0YjlkNmQzLTAxMTItNDBiZi05MTZiLWVlZDFkOGY3NjBhNSIsInQiOmZhbHNlLCJ1aWQiOjE5NjI0NzM2fQ.IEdxZFeC-HddNOUSY3mkzfJi7lp75ox4Xgu9j_yRLdKKNVOGDaxq3PqeA1bACAOth6wjZtYYPI39-T-Y3IT0Pw",
    telegramToken: "7529315847:AAGkdEUV-4uZvsauYev3WAA06YxvGUuYsd0",
    chatId: "-4562130542",
  },
];

const transformedCompanyOptions = computed(() => {
  return companyArray.map(company => ({
    label: company.name,
    value: JSON.stringify(company)
  }));
});

let timerId = null;
const isRunning = ref(false);

const companyOptions = ref([]);

const companySelected = ref(JSON.stringify(companyArray[0]));

companyOptions.value = transformedCompanyOptions.value;

const transformedCompanySelected = computed(() => JSON.parse(companySelected.value));

watch(transformedCompanySelected, () => {
  initValues();
})

const columns =  ref([
  {
    title: 'Время нахождения лимита',
    dataIndex: 'currentDateTime',
    key: 'currentDateTime',
    width: '20%',
  },
  {
    title: 'Дата',
    dataIndex: 'date',
    key: 'date',
    width: '10%',
  },
  {
    title: 'Коэффициент',
    dataIndex: 'coefficient',
    key: 'coefficient',
    width: '05%',
  },
  {
    title: 'Склад',
    dataIndex: 'warehouseName',
    key: 'warehouseName',
    width: '25%',
  },
  {
    title: 'Тип поставки',
    dataIndex: 'boxTypeName',
    key: 'boxTypeName',
    width: '35%',
  },
]);

const columnsData = computed(() => {
  return filteredDataFinish.value.map((dataItem) => ({
    currentDateTime: getCurrentDateTime(),
    date: dayjs(dataItem.date).format('DD.MM.YYYY'),
    coefficient: dataItem.coefficient === 0 ? "Бесплатно" : `x${ dataItem.coefficient }`,
    warehouseName: dataItem.warehouseName,
    boxTypeName: dataItem.boxTypeName,
  }));
});

const deliveryType = ref({
  canBox: false,
  canMonopallet: false,
  canSupersafe: false,
});

function fieldCompanies(fieldName) {
  return `${transformedCompanySelected.value.name.replaceAll(" ", "")}_${fieldName}`
}

watch(deliveryType, (newValue) => {
  localStorage.setItem(fieldCompanies("delivery-type"), JSON.stringify(newValue));
}, { deep: true });

function initValues() {
  const getWarehouses = localStorage.getItem(fieldCompanies("warehouses"));
  warehousesSelected.value = JSON.parse(getWarehouses) || [];

  const getNomenclature = localStorage.getItem(fieldCompanies("nomenclature"));
  nomenclatureSelected.value = JSON.parse(getNomenclature) || [];

  const getRemoveSC = localStorage.getItem(fieldCompanies("sc"));
  removeSC.value = getRemoveSC === "true";

  const getDeliveryType = localStorage.getItem(fieldCompanies("delivery-type"));
  deliveryType.value = JSON.parse(getDeliveryType) || deliveryType.value;

  const getNumberPosition = localStorage.getItem(fieldCompanies("number-positions"));
  numberPositions.value = JSON.parse(getNumberPosition) || 1;

  const getCoefficientFrom = localStorage.getItem(fieldCompanies("coefficient-from"));
  coefficientFrom.value = JSON.parse(getCoefficientFrom) || 0;

  const getCoefficientTo = localStorage.getItem(fieldCompanies("coefficient-to"));
  coefficientTo.value = JSON.parse(getCoefficientTo) || 2;
}

onMounted(() => {
  initValues();
});

// СПИСОК СКЛАДОВ, КОТОРЫЙ ПРИШЕЛ ИЗ API
const warehousesOptions = ref([]);

// СПИСОК СКЛАДОВ, КОТОРЫЕ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const warehousesSelected = ref([]);

watch(warehousesSelected, (newValue) => {
  localStorage.setItem(fieldCompanies("warehouses"), JSON.stringify(newValue));
});

// СПИСОК НОМЕНКЛАТУР, КОТОРЫЙ ПРИШЕЛ ИЗ API
const nomenclatureOptions = ref([]);

// СПИСОК НОМЕНКЛАТУР, КОТОРЫЕ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const nomenclatureSelected = ref([]);

watch(nomenclatureSelected, (newValue) => {
  localStorage.setItem(fieldCompanies("nomenclature"), JSON.stringify(newValue));
});

const removeSC = ref(false);

watch(removeSC, (newValue) => {
  localStorage.setItem(fieldCompanies("sc"), newValue);
});

// КОЛИЧЕСТВО ПОЗИЦИЙ
const numberPositions = ref(1);

watch(numberPositions, (newValue) => {
  localStorage.setItem(fieldCompanies("number-positions"), newValue);
});

// КОЭФФИЦИЕНТ ОТ
const coefficientFrom = ref(0);

watch(coefficientFrom, (newValue) => {
  localStorage.setItem(fieldCompanies("coefficient-from"), newValue);
});

// КОЭФФИЦИЕНТ ДО
const coefficientTo = ref(2);

watch(coefficientTo, (newValue) => {
  localStorage.setItem(fieldCompanies("coefficient-to"), newValue);
});

const matchedWarehouseIDs = ref([]);
const unmatchedWarehouseIDs = ref([]);

const startingDate = ref(dayjs().utc().startOf('day'));

const filteredDataFinish = ref([]);

const getCurrentDateTime = () => {
  return new Date().toLocaleTimeString(navigator.language);
};

// РУЗУЛЬТАТ ФИЛЬТРА, СКЛАДЫ КОТОРЫЕ ПОДХОДЯТ ДЛЯ ТОВАРА, С ТИПОМ УПАКОВКИ КОРОБ И МОНОПАЛЛЕТ
const warehousesGoodsTypePackagingBoxMonopallets = ref([]);

const presets = ref([
  { label: 'Сегодня', value: dayjs().utc().add(0, 'day').startOf('day') },
  { label: 'Следующая неделя', value: dayjs().utc().add(7, 'day').startOf('day') },
]);

// ФОРМАТ ОТОБРАЖЕНИЯ ДАТЫ
const dateFormat = "DD.MM.YYYY";

async function sendMessageToTelegram(options, status) {
  const { date, coefficient, warehouseName, boxTypeName } = options;

  // мой бот
  // const telegramToken = "7352486646:AAEiy58pLIrwIDqdndfp0qB2sCv07wMviSs";
  // const chatId = "514186798";

  // бот Артёма
  // const telegramToken = "7397979520:AAFH3aY5u-PO9OOegXDl_5hvv1PLUp_3eQ4";
  // const chatId = "428444661";

  // // отправка в мою группу
  // const telegramToken = "7352486646:AAEiy58pLIrwIDqdndfp0qB2sCv07wMviSs";
  // const chatId = "-4518448769";

  // отправка в группу Артёма
  // const telegramToken = "7397979520:AAFH3aY5u-PO9OOegXDl_5hvv1PLUp_3eQ4";
  // const chatId = "-4587468459";

  const url = `https://api.telegram.org/bot${transformedCompanySelected.value.telegramToken}/sendMessage`;

  const acceptance = coefficient === 0 ? "Бесплатно" : `x${ coefficient }`;

const formattedMessage = `
${ status ? '*Лимит найден 🟢*' : '*Лимит удалён 🔴*' }
${ getCurrentDateTime() }

*Дата:* ${ dayjs(date).format('DD.MM.YYYY') }
*Поставка:* ${ warehouseName }, ${ boxTypeName }
*Приёмка:* ${ acceptance }
`;

  try {
    await axios.post(url, {
      chat_id: transformedCompanySelected.value.chatId,
      text: formattedMessage.trim(),
      parse_mode: 'Markdown'
    });
    // alert('Сообщение отправлено!');
  } catch (error) {
    console.error('Ошибка отправки сообщения:', error);
  }
}

function coefficientsGet() {
  const idsArray = matchedWarehouseIDs.value.map(warehouse => warehouse.ID);
  const paramsStr = idsArray.join(',');

  // ПОЛУЧАЕМ ВЕСЬ СПИСОК СКЛАДОВ (БЕЗ ПАРАМЕТРОВ)
  axios.get("https://supplies-api.wildberries.ru/api/v1/acceptance/coefficients", {
    params: {
      warehouseIDs: paramsStr
    },
    headers: {
      "Authorization": `${transformedCompanySelected.value.apiToken}`
    }
  })
    .then(response => {
      const filteredData = response.data.filter(item => {
        // Сравнение date с датами из массива dates
        const isDateValid = dayjs(item.date) >= dayjs(startingDate.value);

        // Сравнение coefficient с coefficientFrom и coefficientTo
        const isCoefficientValid = item.coefficient >= coefficientFrom.value && item.coefficient <= coefficientTo.value;

        // Сравнение с boxTypeName
        const isBoxTypeNameValid = (deliveryType.value.canBox && item.boxTypeName === "Короба") || (deliveryType.value.canMonopallet && item.boxTypeName === "Монопаллеты")

        // Возврат элемента, который соответствует всем условиям
        return isDateValid && isCoefficientValid && isBoxTypeNameValid;
      });

      // Вывод отфильтрованных элементов
      filteredDataFinish.value = filteredData;
    })
    .catch(error => {
      console.log(error);
    });
}

watch(filteredDataFinish, (newArray, oldArray) => {
  // Поиск отсутствующего элемента
  for (let i = 0; i < oldArray.length; i++) {
    if (!newArray.find(item => JSON.stringify(item) === JSON.stringify(oldArray[i]))) {

      sendMessageToTelegram(oldArray[i], false);
    }
  }

  // Проверка и вывод новых изменений
  for (let i = 0; i < newArray.length; i++) {
    if (JSON.stringify(newArray[i]) !== JSON.stringify(oldArray[i])) {

      sendMessageToTelegram(newArray[i], true);
    }
  }
});


// ПРИ НАЖАТИИ НА КНОПКУ ВЫБРАТЬ ВСЕ СКЛАДЫ, В warehousesSelected ДОБАВЛЯЕМ СКЛАДЫ
function selectAllWarehouses() {
  const destructuringWarehousesOptions = warehousesOptions.value.map((warehouse) => warehouse.value)

  warehousesSelected.value = [...destructuringWarehousesOptions];
}

// ПОЛУЧАЕМ СКЛАДЫ ПО БАРКОДУ (С ПАРАМЕТРАМИ: КОЛИЧЕСТВО И БАРКОД ТОВАРА)
function receivingWarehousesBarcode(arrayGoods) {
  axios.post("https://supplies-api.wildberries.ru/api/v1/acceptance/options", arrayGoods, {
    headers: {
      "Authorization": `${transformedCompanySelected.value.apiToken}`
    }
  })
    .then(response => {
      // ФИЛЬТРУЕМ СКЛАДЫ КОТОРЫЕ ПОДХОДЯТ ДЛЯ ТОВАРА, ОСТАВЛЯЕМ ГДЕ ТИП УПАКОВКИ КОРОБ И МОНОПАЛЛЕТ
      const filteredWarehouses = [];

      response.data.result.forEach(result => {
        filteredWarehouses.push(...result.warehouses.filter(warehouse => {
          return (deliveryType.value.canBox && warehouse.canBox) || (deliveryType.value.canMonopallet && warehouse.canMonopallet);
        }));
      });

      warehousesGoodsTypePackagingBoxMonopallets.value = filteredWarehouses;

      const tempMatchedWarehouseIDs = [];
      const tempUnmatchedWarehouseIDs = [];

      // СРАВНИВАЕМ СКЛАДЫ С ТЕМ ЧТО ВВЁЛ ПОЛЬЗОВАТЕЛЬ И ДОБАВЛЯЕМ В МАССИВЫ matchedWarehouseIDs И unmatchedWarehouseIDs
      warehousesSelected.value.forEach(warehouseSelected => {
        const parsedWarehouse = JSON.parse(warehouseSelected);

        // НА ДАННОМ ЭТАПЕ, ЕСТЬ МАССИВ parsedWarehouse И warehousesGoodsTypePackagingBoxMonopallets

        const foundMatch = warehousesGoodsTypePackagingBoxMonopallets.value.find(warehouse => {
          if (removeSC.value && parsedWarehouse.name.toLowerCase().startsWith('сц ')) {
            return false; // если условие выполняется, возвращаем false
          }

          return parsedWarehouse.ID === warehouse.warehouseID;
        });

        if (foundMatch) {
          tempMatchedWarehouseIDs.push(parsedWarehouse);
        } else {
          tempUnmatchedWarehouseIDs.push(parsedWarehouse);
        }
      });

      matchedWarehouseIDs.value = tempMatchedWarehouseIDs;
      unmatchedWarehouseIDs.value = tempUnmatchedWarehouseIDs;

      coefficientsGet();
    })
    .catch(error => {
      console.log(error);
    });
}

function startFilters() {
  warehousesGoodsTypePackagingBoxMonopallets.value = [];
  matchedWarehouseIDs.value = [];
  unmatchedWarehouseIDs.value = [];

  const arrayGoods = nomenclatureSelected.value
    .map(nomenclature => JSON.parse(nomenclature))
    .flatMap(nomenclatureObject => nomenclatureObject.sizes ?? [])
    .flatMap(sizeItem => sizeItem.skus ?? [])
    .map(skusItem => ({
      quantity: numberPositions.value,
      barcode: skusItem
    }));

  receivingWarehousesBarcode(arrayGoods);
}

const handleStart = () => {
  isRunning.value = true;
  startFilters();
  timerId = setInterval(startFilters, 11500); // Вызывать функцию каждые 10 секунд
};

const handleStop = () => {
  isRunning.value = false;
  clearInterval(timerId); // Остановить таймер
};

const onDateChange = (date) => {
  if (date) {
    startingDate.value = date.startOf('day');
  }
};

// ПОЛУЧАЕМ ВЕСЬ СПИСОК СКЛАДОВ (БЕЗ ПАРАМЕТРОВ)
watch(transformedCompanySelected, (newValue) => {
  axios.get("https://supplies-api.wildberries.ru/api/v1/warehouses", {
    headers: {
      "Authorization": `${newValue.apiToken}`
    }
  })
    .then(response => {
      const transformData = response.data.map((warehouse) => ({
        ID: warehouse.ID,
        name: warehouse.name,
      }));

      warehousesOptions.value = transformData.map((warehouse) => ({
        label: warehouse.name,
        value: JSON.stringify(warehouse),
      }));
    })
    .catch(error => {
      console.log(error);
    });

  // ПОЛУЧАЕМ ВЕСЬ СПИСОК НОМЕНКЛАТУР
// (С ПАРАМЕТРАМИ: ФИЛЬТР ПО ФОТО - ВСЕ КАРТОЧКИ, ЛИМИТ - СКОЛЬКО КАРТОЧЕК ТОВАРА ВЕРНУТЬ)
  axios.post("https://content-api.wildberries.ru/content/v2/get/cards/list", {
    settings: {
      cursor: {
        limit: 100
      },
      filter: {
        withPhoto: -1
      }
    }
  }, {
    headers: {
      "Authorization": `${newValue.apiToken}`
    }
  })
    .then(response => {
      const transformData = response.data.cards.map((card) => ({
        vendorCode: card.vendorCode,
        sizes: card.sizes,
      }));

      nomenclatureOptions.value = transformData.map((nomenclature) => ({
        label: nomenclature.vendorCode,
        value: JSON.stringify(nomenclature),
      }));
    })
    .catch(error => {
      console.log(error);
    });
}, { immediate: true });
</script>

<template>
  <a-config-provider :locale="ruRu">
    <a-form ref="form" layout="vertical">
      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="Компания" name="companySelected">
            <a-select
              v-model:value="companySelected"
              :options="companyOptions"
              placeholder="Выберите из списка"
              show-search
            />
          </a-form-item>
        </a-col>
      </a-row>

      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="Склад" name="warehousesSelected">
            <a-select
              v-model:value="warehousesSelected"
              :options="warehousesOptions"
              :max-tag-count="10"
              mode="multiple"
              placeholder="Выберите из списка"
              show-search
              allow-clear
            />
          </a-form-item>
        </a-col>

        <a-col :span="3">
          <a-form-item label=" " name="removeSC">
            <a-checkbox v-model:checked="removeSC">
              Убрать СЦ
            </a-checkbox>
          </a-form-item>
        </a-col>

        <a-col :span="9">
          <a-form-item label="Номенклатуры" name="warehousesSelected">
            <a-select
              v-model:value="nomenclatureSelected"
              :options="nomenclatureOptions"
              :max-tag-count="10"
              mode="multiple"
              placeholder="Выберите из списка"
              show-search
              allow-clear
            />
          </a-form-item>
        </a-col>
      </a-row>

      <a-row :gutter="24">
        <a-col :span="4">
          <a-form-item>
            <a-button
              html-type="submit"
              @click="selectAllWarehouses"
            >
              Выбрать все склады
            </a-button>
          </a-form-item>
        </a-col>
      </a-row>

      <a-row :gutter="24">
        <a-col :span="3">
          <a-form-item
            label="Коэффициент от"
            name="warehousesSelected"
          >
            <a-input
              v-model:value.number="coefficientFrom"
              placeholder="Значение от"
            >
            </a-input>
          </a-form-item>
        </a-col>

        <a-col :span="3">
          <a-form-item
            label="Коэффициент до"
            name="warehousesSelected"
          >
            <a-input
              v-model:value.number="coefficientTo"
              placeholder="Значение до"
            >
            </a-input>
          </a-form-item>
        </a-col>

        <a-col :span="6">
          <a-form-item
            label="Количество позиций"
            name="warehousesSelected"
          >
            <a-input
              v-model:value.number="numberPositions"
              placeholder="Колличество"
            >
            </a-input>
          </a-form-item>
        </a-col>

        <a-col :span="4">
          <a-form-item label="Начиная с даты" name="warehousesSelected"
          >
            <a-date-picker
              v-model:value="startingDate"
              :format="dateFormat"
              :presets="presets"
              @change="onDateChange"
            />
          </a-form-item>
        </a-col>

        <a-col :span="3">
          <a-form-item label="Тип поставки" name="canBox">
            <a-checkbox v-model:checked="deliveryType.canBox">
              Короб
            </a-checkbox>
          </a-form-item>
        </a-col>

        <a-col :span="3">
          <a-form-item label="Тип поставки" name="canMonopallet">
            <a-checkbox v-model:checked="deliveryType.canMonopallet">
              Монопаллет
            </a-checkbox>
          </a-form-item>
        </a-col>
      </a-row>

      <a-row>
        <a-col :span="3">
          <a-form-item>
            <a-button
              type="primary"
              size="large"
              html-type="submit"
              :disabled="isRunning"
              @click="handleStart"
            >
              Запустить
            </a-button>
          </a-form-item>
        </a-col>

        <a-col :span="3">
          <a-form-item>
            <a-button
              size="large"
              html-type="submit"
              :disabled="!isRunning"
              @click="handleStop"
            >
              Остановить
            </a-button>
          </a-form-item>
        </a-col>
      </a-row>
    </a-form>

    <a-table :data-source="columnsData" :columns="columns" />
  </a-config-provider>
</template>

<style scoped></style>
