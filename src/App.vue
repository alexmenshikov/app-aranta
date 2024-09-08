<script setup>
import { computed, ref, watch } from "vue";
import axios from "axios";
import { ConfigProvider as AConfigProvider } from "ant-design-vue";
import ruRu from "ant-design-vue/es/locale/ru_RU";
import dayjs from "dayjs";
import ru from "dayjs/locale/ru";
dayjs.locale(ru);

// ТОКЕН ДЛЯ ДОСТУПА К API
const apiToken = "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjQwOTA0djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc0MTI5ODI5MywiaWQiOiIyOGUxZGRlNS1kMzU5LTQ4MDktYTY3OC04ZTdkNDdmZWIyNzgiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwMTg1MzQsInMiOjEwMjYsInNpZCI6ImRjZTNhNzQ5LWU0ZmQtNDkwMC1iYmYyLWJjMzYyODNkOTk4MCIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.ibhHx9zTX066nuHD6dBoTcf0tK3q0_Tv6vhpHFbk6-qmpgBAKmuP1_NXkCTe1vFqINILROWWj6Qx925YPlSUgg";

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
  const futureDateFirst = new Date(currentDate);
  const futureDateSecond = new Date(currentDate);

  futureDateFirst.setDate(currentDate.getDate() + from);
  futureDateSecond.setDate(currentDate.getDate() + to);

  return [
    dayjs(futureDateFirst),
    dayjs(futureDateSecond)
  ]
}

// СБРАСЫВАЕТ СЕЛЕКТ С ДАТАМИ, ЕСЛИ ПОЛЬЗОВАТЕЛЬ, ВЫБРАЛ СВОЙ ВАРИАНТ
const handleChange = (dates) => {
  datesSelected.value = null;
}

// ФОРМАТ ОТОБРАЖЕНИЯ ДАТЫ
const dateFormat = "DD.MM.YYYY";


// function getDateFormat(withTime = true) {
//   if (!withTime) {
//     return "DD.MM.YY";
//   }
//   return "DD.MM.YY HH:mm:ss";
// }

// function selectAllWarehouses() {
//   warehousesSelected.value = warehousesOptions.value.map(warehousesOption => warehousesOption.value);
// }

// function warehousesOptionsAllInfoSelected() {
//   console.log('~ warehousesOptions: ', warehousesOptions.value);
//   console.log('~ warehousesSelected: ', warehousesSelected.value);
//
//   return warehousesOptions.value.filter(obj => {
//     warehousesSelected.value.includes(obj.value);
//   })
// }

// +++++++++++++++
// function warehousesOptionsAllInfoSelected() {
//   return warehousesOptions.value.filter(obj => {
//     return warehousesSelected.value.includes(obj.value);
//   })
// }
// +++++++++++++++

// const warehousesOptionsAllInfoSelected = ref([]);

// watch(warehousesSelected, (newWarehousesSelected) => {
//   warehousesOptionsAllInfoSelected.value = warehousesOptions.value.filter((obj) => {
//     newWarehousesSelected.value.includes(obj.value);
//   })
// })

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
      const finishFilterResult = [];

      // response.data.forEach(element => {
      //   finishFilterResult.push()
      // })

      const filteredData = response.data.filter(item => {
        // Сравнение date с датами из массива dates
        const isDateValid = dayjs(item.date) > dayjs(dates.value[0]) && dayjs(item.date) < dayjs(dates.value[1]);

        // Сравнение coefficient с coefficientFrom и coefficientTo
        const isCoefficientValid = item.coefficient >= coefficientFrom.value && item.coefficient <= coefficientTo.value;

        // Проверка условий для deliveryType
        // const isDeliveryTypeValid = (deliveryType.value.canBox && item.canBox) || (deliveryType.value.canMonopallet && item.canMonopallet);

        // Сравнение с boxTypeName
        const isBoxTypeNameValid = (deliveryType.value.canBox && item.boxTypeName === "Короба") || (deliveryType.value.canMonopallet && item.boxTypeName === "Монопаллеты")

        // Возврат элемента, который соответствует всем условиям
        // return isDateValid && isCoefficientValid && isBoxTypeNameValid;
        return isDateValid && isCoefficientValid && isBoxTypeNameValid;
      });

      // Вывод отфильтрованных элементов
      // console.log("filteredData: ", filteredData);
      filteredDataFinish.value = filteredData;
    })
    .catch(error => {
      // Обработка ошибки
      console.log(error);
    });
}

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

  // СПИСОК С ПОДХОДЯЩИМИ СКЛАДАМИ
  // matchedWarehouseIDs.value

  // axios.get("https://supplies-api.wildberries.ru/api/v1/acceptance/coefficients", { params }, {
  //   headers: {
  //     "Authorization": `${apiToken}`
  //   }
  // })
  //   .then(response => {
  //     console.log("Ответ: ", response.data);
  //     // warehousesOptions.value = response.data.map((warehouse) => ({
  //     //   label: warehouse.name,
  //     //   value: JSON.stringify(warehouse),
  //     // }));
  //   })
  //   .catch(error => {
  //     console.log(error);
  //   });
}

function startFilters() {
  console.log("startFilters");

  warehousesGoodsTypePackagingBoxMonopallets.value = [];
  matchedWarehouseIDs.value = [];
  unmatchedWarehouseIDs.value = [];
  // if (warehousesSelected.value.length > 0 && nomenclatureSelected.value.length > 0 && numberPositions.value && coefficientFrom.value && coefficientTo.value) {
  // if (warehousesSelected.value.length > 0 && nomenclatureSelected.value.length > 0) {
  //   const arrayGoods = [];
  //
  //   nomenclatureSelected.value
  //     .map(nomenclature => JSON.parse(nomenclature))
  //     .flatMap(nomenclatureObject => nomenclatureObject.sizes ?? [])
  //     .flatMap(sizeItem => sizeItem.skus ?? [])
  //     .forEach(skusItem => {
  //       arrayGoods.push({
  //         quantity: numberPositions.value,
  //         barcode: skusItem
  //       });
  //     });
  const arrayGoods = nomenclatureSelected.value
    .map(nomenclature => JSON.parse(nomenclature))
    .flatMap(nomenclatureObject => nomenclatureObject.sizes ?? [])
    .flatMap(sizeItem => sizeItem.skus ?? [])
    .map(skusItem => ({
      quantity: numberPositions.value,
      barcode: skusItem
    }));

  receivingWarehousesBarcode(arrayGoods);
  // }
  // const barcodesSelectedItems = ref([]);

  // nomenclatureSelected.value.forEach(nomenclature => {
  //   // barcodesSelectedItems.value = nomenclatureOptions.value.filter(option => option.value === nomenclature);
  //   barcodesSelectedItems.value = nomenclatureOptions.value.filter(option => option.value === nomenclature);
  // })

  // barcodesSelectedItems.value = nomenclatureSelected.value.map(nomenclature)


  // barcodesSelectedItems.value = nomenclatureOptions.value
  //   .filter(obj => nomenclatureSelected.value.includes(obj.value)) // Фильтруем объекты по совпадению значения 'name'
  //   .map(obj => obj.infoWithApi.sizes[0].skus[0]); // Извлекаем значения 'barcode' для соответствующих объектов

  // лежат баркоды в barcodesSelectedItems.value
  // console.log('barcodesSelectedItems:', barcodesSelectedItems.value);

  // const dataArray = [{
  //   quantity: 1,
  //   barcode: "2040581715663"
  // }];



  //   axios.post("https://supplies-api.wildberries.ru/api/v1/acceptance/options", dataArray, {
  //     headers: {
  //       "Authorization": `${apiToken}`
  //     }
  //   })
  //     .then(response => {
  //       console.log("response: ", response.data.result[0].warehouses);
  //
  //       // получили все склады где есть подходящие типы
  //       warehousesFiltered.value = response.data.result[0].warehouses.filter(warehouse => warehouse.canBox || warehouse.canMonopallet);
  //
  //       // const tmp = warehousesOptionsAllInfoSelected();
  //       // console.log('warehousesOptionsAllInfoSelected', tmp);
  //
  //       console.log("----: ", warehousesFiltered.value);
  //
  //       warehousesFiltered.value.forEach((warehouse1) => {
  //         const foundMatch = warehousesOptionsAllInfoSelected().find(warehouse2 => warehouse1.warehouseID === warehouse2.infoWithApi.ID);
  //
  //         if (foundMatch) {
  //           matchedWarehouseIDs.value.push(warehouse1);
  //         } else {
  //           unmatchedWarehouseIDs.value.push(warehouse1);
  //         }
  //       })
  //
  //
  //
  //       // matchedWarehouseIDs
  //       // unmatchedWarehouseIDs
  //
  //       // warehousesFiltered.value = response.data.warehouses
  //       //   .map(warehouse => {
  //       //
  //       //   })
  //
  //       // nomenclatureOptions.value = response.data.cards.map((nomenclature) => ({
  //       //   infoWithApi: nomenclature,
  //       //   label: nomenclature.vendorCode,
  //       //   value: nomenclature.vendorCode,
  //       // }));
  //     })
  //     .catch(error => {
  //       console.log(error);
  //     });
  // }
}

// function warehousesChange(event) {
//   if (event) {
//     event.forEach(item => {
//       console.log(item);
//     })
//   }
// }

// const setCopySiteOffersFromFinShowcaseSiteId = (event) => {
//   emit("update:copySiteOffersFromFinShowcaseSiteId", event);
// };

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

        <a-col :span="12">
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
        <a-col>
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

        <a-col :span="3">
          <a-form-item label=" " name="removeSC">
            <a-checkbox v-model:checked="removeSC">
              Убрать СЦ
            </a-checkbox>
          </a-form-item>
        </a-col>

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
        <a-col>
          <a-form-item>
            <a-button
              type="primary"
              size="large"
              html-type="submit"
              @click="startFilters"
            >
              Сохранить
            </a-button>
          </a-form-item>
        </a-col>
      </a-row>
    </a-form>

    <div v-if="filteredDataFinish.length > 0">
      <b>Подходят склады:</b>
      <div v-for="(item, index) in filteredDataFinish" :key="index">
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
