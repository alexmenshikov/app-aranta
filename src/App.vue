<script setup>
import { ref, watch } from "vue";
import axios from "axios";
import { ConfigProvider as AConfigProvider } from "ant-design-vue";
import ruRu from "ant-design-vue/es/locale/ru_RU";
import dayjs from "dayjs";
import ru from "dayjs/locale/ru";
dayjs.locale(ru);

let timerId = null;
const isRunning = ref(false);

// ТОКЕН ДЛЯ ДОСТУПА К API
const apiToken = "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTI5ODI5MywiaWQiOiIyOGUxZGRlNS1kMzU5LTQ4MDktYTY3OC04ZTdkNDdmZWIyNzgiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwMTg1MzQsInMiOjEwMjYsInNpZCI6ImRjZTNhNzQ5LWU0ZmQtNDkwMC1iYmYyLWJjMzYyODNkOTk4MCIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.ibhHx9zTX066nuHD6dBoTcf0tK3q0_Tv6vhpHFbk6-qmpgBAKmuP1_NXkCTe1vFqINILROWWj6Qx925YPlSUgg";

// ТОКЕН ДЛЯ ТЕСТИРОВАНИЯ
// const apiToken = "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTY0MzMwMSwiaWQiOiIwMTkxZDYyZi0yYWFhLTcwN2UtOGMyMS0zZjY1NTczNjMyYTQiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwMTg1MzQsInMiOjEwMjYsInNpZCI6ImRjZTNhNzQ5LWU0ZmQtNDkwMC1iYmYyLWJjMzYyODNkOTk4MCIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.wNkYtKtCq7ekhVxE754sGW-xTOx_MfFBriDkYl_3BG-BRMwtlLXhnhZoOBmVb_WJNMCrBJ3QWiOPh16XsofFNw";

// СПИСОК СКЛАДОВ, КОТОРЫЙ ПРИШЕЛ ИЗ API
const warehousesOptions = ref([]);

// СПИСОК СКЛАДОВ, КОТОРЫЕ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const warehousesSelected = ref([]);

// СПИСОК НОМЕНКЛАТУР, КОТОРЫЙ ПРИШЕЛ ИЗ API
const nomenclatureOptions = ref([]);

// СПИСОК НОМЕНКЛАТУР, КОТОРЫЕ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const nomenclatureSelected = ref([]);

// КОЛИЧЕСТВО ПОЗИЦИЙ
const numberPositions = ref(1);

// КОЭФФИЦИЕНТ ОТ
const coefficientFrom = ref(0);

// КОЭФФИЦИЕНТ ДО
const coefficientTo = ref(2);
// const warehousesFiltered = ref([]);

const matchedWarehouseIDs = ref([]);
const unmatchedWarehouseIDs = ref([]);

// ЗНАЧЕНИЕ ДАТЫ
const dates = ref([]);

// ДАТА, КОТОРУЮ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const datesSelected = ref(null);

const deliveryType = ref({
  canBox: false,
  canMonopallet: false,
  canSupersafe: false,
});

const removeSC = ref(false);

const filteredDataFinish = ref([]);

// const firstArray = ref([]);
// const secondArray = ref([]);
// const thirdArray = ref([]);

const getCurrentDateTime = () => {
  return new Date().toLocaleString();
};

// РУЗУЛЬТАТ ФИЛЬТРА, СКЛАДЫ КОТОРЫЕ ПОДХОДЯТ ДЛЯ ТОВАРА, С ТИПОМ УПАКОВКИ КОРОБ И МОНОПАЛЛЕТ
const warehousesGoodsTypePackagingBoxMonopallets = ref([]);

// ИЗМЕНЕНИИ ДАТЫ, В ЗАВИСИМОСТИ ОТ ВЫБРАННОГО СЕЛЕКТ (14 ДНЕЙ, ПЕРВАЯ НЕДЕЛЯ, ВТОРАЯ НЕДЕЛЯ)
function changeDatesOptions(event) {
  typeof event === "string" ? dates.value = creatingDateRange(JSON.parse(event)) : dates.value = [];
}

// СПИСОК ВАРИАНТОВ ДАТ
const datesOptions = [
  {
    label: "14 дней",
    value: JSON.stringify({ from: 0, to: 14 })
  },
  {
    label: "Первая неделя",
    value: JSON.stringify({ from: 0, to: 7 })
  },
  {
    label: "Вторая неделя",
    value: JSON.stringify({ from: 7, to: 14 })
  }
];

// ФУНКЦИЯ ЗАДАЁТ ЗНАЧЕНИЕ ДАТЫ В ЗАВИСИМОСТИ ОТ ВЫБОРА ПОЛЬЗОВАТЕЛЯ
function creatingDateRange(options) {
  const { from, to } = options;

  const currentDate = new Date();
  const utcTime = Date.UTC(currentDate.getUTCFullYear(), currentDate.getUTCMonth(), currentDate.getUTCDate(), 0, 0, 0, 0);
  const utcCurrentDate = new Date(utcTime);

  const futureDateFirst = new Date(utcCurrentDate);
  const futureDateSecond = new Date(utcCurrentDate);

  futureDateFirst.setUTCDate(utcCurrentDate.getUTCDate() + from);
  futureDateSecond.setUTCDate(utcCurrentDate.getUTCDate() + to);

  return [
    dayjs(futureDateFirst.toISOString()),
    dayjs(futureDateSecond.toISOString())
  ]
}

// СБРАСЫВАЕТ СЕЛЕКТ С ДАТАМИ, ЕСЛИ ПОЛЬЗОВАТЕЛЬ, ВЫБРАЛ СВОЙ ВАРИАНТ
const handleChange = (dates) => {
  datesSelected.value = null;
}

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
  const telegramToken = "7397979520:AAFH3aY5u-PO9OOegXDl_5hvv1PLUp_3eQ4";
  const chatId = "-4587468459";

  const url = `https://api.telegram.org/bot${telegramToken}/sendMessage`;

  const acceptance = coefficient === 0 ? "Бесплатно" : `x${ coefficient }`;

// const formattedMessage = `
// ${ status ? '<span style="color: #e32636">*Лимит найден*</span>' : '*Лимит удалён*' }
// ${ getCurrentDateTime() }
//
// *Дата:* ${ dayjs(date).format('DD.MM.YYYY') }
// *Поставка:* ${ warehouseName }, ${ boxTypeName }
// *Приёмка:* ${ acceptance }
// `;
//   const formattedMessage = `
//     <span style="color: ${ status ? '#e32636' : '#2bae66' };"><b>${ status ? 'Лимит найден' : 'Лимит удалён' }</b></span>
//     <span>${ getCurrentDateTime() }</span>
//     <span><b>Дата:</b> ${ dayjs(date).format('DD.MM.YYYY') }</span>
//     <span><b>Поставка:</b> ${ warehouseName }, ${ boxTypeName }</span>
//     <span><b>Приёмка:</b> ${ acceptance }</span>
//   `;

const formattedMessage = `
${ status ? '*Лимит найден 🟢*' : '*Лимит удалён 🔴*' }
${ getCurrentDateTime() }

*Дата:* ${ dayjs(date).format('DD.MM.YYYY') }
*Поставка:* ${ warehouseName }, ${ boxTypeName }
*Приёмка:* ${ acceptance }
`;

  try {
    await axios.post(url, {
      chat_id: chatId,
      text: formattedMessage.trim(),
      parse_mode: 'Markdown'
    });
    // alert('Сообщение отправлено!');
  } catch (error) {
    console.error('Ошибка отправки сообщения:', error);
  }
}

// function compareArrays(arr1, arr2) {
//   if (JSON.stringify(arr1) !== JSON.stringify(arr2)) {
//     thirdArray.value = ['Arrays are not equal'];
//     firstArray.value = arr2;
//   } else {
//     thirdArray.value = [];
//   }
// }

function coefficientsGet() {
  const idsArray = matchedWarehouseIDs.value.map(warehouse => warehouse.ID);
  const paramsStr = idsArray.join(',');

  // ПОЛУЧАЕМ ВЕСЬ СПИСОК СКЛАДОВ (БЕЗ ПАРАМЕТРОВ)
  axios.get("https://supplies-api.wildberries.ru/api/v1/acceptance/coefficients", {
    params: {
      warehouseIDs: paramsStr
    },
    headers: {
      "Authorization": `${apiToken}`
    }
  })
    .then(response => {
      const filteredData = response.data.filter(item => {
        // Сравнение date с датами из массива dates
        const isDateValid = dayjs(item.date) >= dayjs(dates.value[0]) && dayjs(item.date) <= dayjs(dates.value[1]);

        // Сравнение coefficient с coefficientFrom и coefficientTo
        const isCoefficientValid = item.coefficient >= coefficientFrom.value && item.coefficient <= coefficientTo.value;

        // Сравнение с boxTypeName
        const isBoxTypeNameValid = (deliveryType.value.canBox && item.boxTypeName === "Короба") || (deliveryType.value.canMonopallet && item.boxTypeName === "Монопаллеты")

        // Возврат элемента, который соответствует всем условиям
        return isDateValid && isCoefficientValid && isBoxTypeNameValid;
      });

      // let sortedArray = filteredData.sort((a, b) => a.coefficient - b.coefficient);

      // let sortedArray = filteredData.sort((a, b) => {
      //   if (a.coefficient === b.coefficient) {
      //     return new Date(a.date) - new Date(b.date);
      //   }
      //   return a.coefficient - b.coefficient;
      // });


      // Вывод отфильтрованных элементов
      filteredDataFinish.value = filteredData;
    })
    .catch(error => {
      console.log(error);
    });



  // const firstArray = ref([]);
  // const secondArray = ref([]);
  // const thirdArray = ref([]);
  // filteredDataFinish.value.forEach(item => {
  //   sendMessageToTelegram(item);
  // })
}

// watch(filteredDataFinish, (newVal, oldVal) => {
//   const addedItems = newVal.filter(item => !oldVal.includes(item));
//   if (addedItems.length > 0) {
//     console.log('Добавленные элементы:', addedItems);
//   }
// }, { deep: true });

// watch(filteredDataFinish, (newVal, oldVal) => {
//   if (oldVal.length === 0) return; // Пропускаем начальную инициализацию
//   const addedItems = newVal.filter(item => !oldVal.includes(item));
//   if (addedItems.length > 0) {
//     console.log('Добавленные элементы:', addedItems);
//   }
// }, { deep: true });

watch(filteredDataFinish, (newArray, oldArray) => {
  // console.log('Старый массив:', oldArray);
  // console.log('Новый массив:', newArray);

  // Поиск отсутствующего элемента
  for (let i = 0; i < oldArray.length; i++) {
    if (!newArray.find(item => JSON.stringify(item) === JSON.stringify(oldArray[i]))) {
      // console.log(Исчез элемент из массива:, oldArray[i]);
      sendMessageToTelegram(oldArray[i], false);
    }
  }

  // Проверка и вывод новых изменений
  for (let i = 0; i < newArray.length; i++) {
    if (JSON.stringify(newArray[i]) !== JSON.stringify(oldArray[i])) {
      // console.log(`Изменения в элементе с индексом ${i}:, newArray[i]`);
      // console.log(`Изменения ${newArray[i]}`);
      // console.log(`Изменения ${JSON.stringify(newArray[i], null, 2)}`);
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
      "Authorization": `${apiToken}`
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
  timerId = setInterval(startFilters, 10500); // Вызывать функцию каждые 10 секунд
};

const handleStop = () => {
  isRunning.value = false;
  clearInterval(timerId); // Остановить таймер
};


// ПОЛУЧАЕМ ВЕСЬ СПИСОК СКЛАДОВ (БЕЗ ПАРАМЕТРОВ)
axios.get("https://supplies-api.wildberries.ru/api/v1/warehouses", {
  headers: {
    "Authorization": `${apiToken}`
  }
})
  .then(response => {
    warehousesOptions.value = response.data.map((warehouse) => ({
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
    "Authorization": `${apiToken}`
  }
})
  .then(response => {
    nomenclatureOptions.value = response.data.cards.map((nomenclature) => ({
      label: nomenclature.vendorCode,
      value: JSON.stringify(nomenclature),
    }));
  })
  .catch(error => {
    console.log(error);
  });
</script>

<template>
  <a-config-provider :locale="ruRu">
    <a-form ref="form" layout="vertical">
      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="Склады" name="warehousesSelected">
            <a-select
              v-model:value="warehousesSelected"
              :options="warehousesOptions"
              :max-tag-count="10"
              mode="multiple"
              placeholder="Выберите из списка"
              show-search
              allow-clear
            >
            </a-select>
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
            >
            </a-select>
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
      <a-row :gutter="24">
        <a-col :span="6">
          <a-form-item label="Выбор недель" name="warehousesSelected"
          >
            <a-select
              v-model:value="datesSelected"
              :options="datesOptions"
              @change="changeDatesOptions"
              placeholder="Выберите из списка"
              allow-clear
            >
            </a-select>
          </a-form-item>
        </a-col>

        <a-col :span="6">
          <a-form-item label="Даты" name="warehousesSelected"
          >
            <a-range-picker
              v-model:value="dates"
              :format="dateFormat"
              @change="handleChange"
            />
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

    <div v-if="filteredDataFinish.length > 0">
      <b>Подходят склады:</b>
      <div v-for="(item, index) in filteredDataFinish" :key="index"
           :style="{ 'background-color': item.coefficient === 0 ? '#cafcca' : (item.coefficient === 1 ? '#fafad2' : '') }"
      >
        Время: {{ getCurrentDateTime() }} - Коэффициент: {{ item.coefficient }} - Дата: {{ dayjs(item.date).format('DD.MM.YYYY') }} - Склад: {{item.warehouseName }} - Поставка: {{ item.boxTypeName }}
      </div>
    </div>

    <hr v-if="matchedWarehouseIDs.length > 0 && unmatchedWarehouseIDs.length > 0">
    <div v-if="unmatchedWarehouseIDs.length > 0">
      <b>Неподходят склады:</b>
      <div v-for="(item, index) in unmatchedWarehouseIDs" :key="item.ID">
        {{ index+1 }}: {{ item.name }}
      </div>
    </div>
  </a-config-provider>
</template>

<style scoped></style>
