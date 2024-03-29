<template>
  <v-app>
    <div class="content">
      <div class="text-h4 ma-4">Заказы</div>
        <v-row>
          <v-col cols="6">
            <v-menu
              v-model="isDateFromOpened"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="dateFrom"
                  label="С"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="dateFrom"
                @input="isDateFromOpened = false"
                no-title
                locale="ru-ru"
                :first-day-of-week="1"
              ></v-date-picker>
            </v-menu>
          </v-col>
          <v-col
            cols="6"
          >
            <v-menu
              v-model="isDateToOpened"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="dateTo"
                  label="По"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="dateTo"
                @input="isDateToOpened = false"
                no-title
                locale="ru-ru"
                :first-day-of-week="1"
              ></v-date-picker>
            </v-menu>
          </v-col>
        </v-row>
        <div>Фильтры заказов за выбранный период времени</div>
        <div class="d-flex justify-space-around align-center filters" style="gap: 5px 20px; flex-wrap: wrap">
          <v-select
            :items="filters.numbers"
            label="Номера заказов"
            outlined
            v-model="numbersFilter"
            chips
            multiple
            :menu-props="{ down: true, offsetY: true }"
            style="width: 280px"
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index === 0">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 1"
                class="grey--text text-caption"
              >
                (+{{ numbersFilter.length - 1 }} others)
              </span>
            </template>          
          </v-select>
          <v-select
            :items="filters.itemNames"
            label="Наименования товаров"
            outlined
            v-model="itemNamesFilter"
            chips
            multiple
            :menu-props="{ down: true, offsetY: true }"
            style="width: 280px"
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index === 0">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 1"
                class="grey--text text-caption"
              >
                (+{{ itemNamesFilter.length - 1 }} others)
              </span>
            </template>                 
          </v-select>
          <v-select
            :items="filters.itemUnits"
            label="Единицы товаров"
            outlined
            v-model="itemUnitsFilter"
            chips
            multiple
            :menu-props="{ down: true, offsetY: true }"
            style="width: 280px"
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index === 0">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 1"
                class="grey--text text-caption"
              >
                (+{{ itemUnitsFilter.length - 1 }} others)
              </span>
            </template>          
          </v-select>
          <v-select
            :items="filters.providerNames"
            label="Наименования поставщиков"
            outlined
            v-model="providerNamesFilter"
            chips
            multiple
            :menu-props="{ down: true, offsetY: true }"
            style="width: 280px"
          >
            <template v-slot:selection="{ item, index }">
              <v-chip v-if="index === 0">
                <span>{{ item }}</span>
              </v-chip>
              <span
                v-if="index === 1"
                class="grey--text text-caption"
              >
                (+{{ providerNamesFilter.length - 1 }} others)
              </span>
            </template>                
          </v-select>
        </div>
        <div class="d-flex justify-center align-center" style="gap: 50px">
          <v-btn elevation="2" @click="getOrders">Получить</v-btn>
          <v-btn elevation="2" @click="openCreationPopup">Создать</v-btn>
      </div>
    </div>
    <v-data-table
      :headers="headers"
      :items="orders"
      :items-per-page="5"
      class="elevation-1 row-pointer"
      @click:row="handleRowClick"
      :loading="isLoadingActive"
      loading-text="Загружаю. Подождите"
      no-data-text="Нет данных"
    >
    </v-data-table>
    <template>
      <div class="text-center">
        <v-dialog
          v-model="isGetDialogOpened"
          width="600"
        >
          <v-card>
            <v-card-title class="text-h5 grey lighten-2">
              Заказ № '{{selectedRow.number}}' от {{selectedOrderDate}}, поставщик: '{{selectedProviderName}}'
            </v-card-title>
            <template>
              <v-simple-table
                fixed-header
                height="300px"
              >
                <template v-slot:default>
                  <thead>
                    <tr>
                      <th class="text-center">Имя</th>
                      <th class="text-center">Количество</th>
                      <th class="text-center">Единица</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in orderItems" :key="item.id">
                      <td class="text-center">{{ item.name }}</td>
                      <td class="text-center">{{ item.quantity }}</td>
                      <td class="text-center">{{ item.unit }}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
              <v-row class="ma-0">
                <v-col cols="6" class="d-flex justify-center">
                  <v-btn small color="primary" elevation="5" @click="editOrder">Редактировать</v-btn>
                </v-col>
                <v-col cols="6" class="d-flex justify-center">
                  <v-btn small color="error" elevation="5" @click="removeOrder(selectedRow.id)">Удалить</v-btn>
                </v-col>
              </v-row>
            </template>
          </v-card>
        </v-dialog>
      </div>
    </template>
    <template>
      <v-dialog
        v-model="isCreationDialogOpened"
        width="1000"
      >
        <v-card ref="form" class="pa-4">
          <v-flex row class="justify-space-around ma-0" style="gap: 10px">
            <v-text-field
              ref="number"
              v-model="number"
              :rules="[() => !!number || 'Обязательное поле']"
              :error-messages="errorMessages"
              label="Номер заказа"
              required
            ></v-text-field>
            <v-menu
              v-model="isOrderDateOpened"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="orderDate"
                  label="Дата заказа"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="orderDate"
                @input="isOrderDateOpened = false"
                no-title
                locale="ru-ru"
                :first-day-of-week="1"
              ></v-date-picker>
            </v-menu>
            <v-menu
              ref="timeMenu"
              v-model="isTimePickerOpened"
              :close-on-content-click="false"
              :nudge-right="40"
              :return-value.sync="newOrderTime"
              transition="scale-transition"
              offset-y
              max-width="290px"
              min-width="290px"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="newOrderTime"
                  label="Время заказа"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-time-picker
                v-if="isTimePickerOpened"
                v-model="newOrderTime"
                full-width
                @click:minute="$refs.timeMenu.save(newOrderTime)"
                format="24hr"
                no-title  
              ></v-time-picker>
            </v-menu>
            <v-autocomplete
              ref="providerRef"
              v-model="provider"
              :rules="[() => !!provider || 'Обязательное поле']"
              :items="providers"
              item-text="name"
              item-value="id"
              label="Поставщик"
              placeholder="Начните вводить название или выберите из списка"
              required
            ></v-autocomplete>
          </v-flex>
          <v-divider></v-divider>
          <div class="d-flex justify-center ma-4">
            <v-btn @click="addOrderItem" small>Добавить товар</v-btn>
          </div>
          <div v-for="(item, index) in newOrderItems" :key="index">
            <v-flex row class="justify-space-around ma-0">
              <v-text-field
                ref="orderItemNameRef"
                v-model="newOrderItems[index].name"
                :rules="[() => !!newOrderItems[index].name || 'Обязательное поле']"
                :error-messages="errorMessages"
                label="Наименование товара"
                required
                style="width: 200px"
              ></v-text-field>
              <v-text-field
                class="col-3"
                ref="orderItemUqantityRef"
                v-model="newOrderItems[index].quantity"
                :rules="[
                  () => !!newOrderItems[index].quantity || 'Обязательное поле',
                  () => !!newOrderItems[index].quantity && /^-?\d+(\.\d+)?$/.test(newOrderItems[index].quantity) || 'Введите число. Пример: 22.2',
                ]"
                :error-messages="errorMessages"
                label="Количество товара"
                required
                style="width: 200px"
              ></v-text-field>
              <v-text-field
                ref="orderItemUnitRef"
                v-model="newOrderItems[index].unit"
                :rules="[() => !!newOrderItems[index].unit || 'Обязательное поле']"
                :error-messages="errorMessages"
                label="Единицы измерения"
                required
                style="width: 200px"
              ></v-text-field>
              <div v-if="newOrderItems.length > 1" class="d-flex justify-center align-center flex-none" style="width: 200px">
                <v-btn small @click="deleteOrderItem(index)" class="ma-4" style="width: 150px">Удалить товар</v-btn>
              </div>
            </v-flex>
          </div>
          <v-divider></v-divider>
          <v-card-actions>
            <v-btn text @click="isCreationDialogOpened=false">Отмена</v-btn>
            <v-spacer></v-spacer>
            <v-slide-x-reverse-transition>
              <v-tooltip
                left
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    icon
                    class="my-0"
                    v-bind="attrs"
                    @click="resetForm"
                    v-on="on"
                  >
                    <v-icon>mdi-refresh</v-icon>
                  </v-btn>
                </template>
                <span>Очистить заказ</span>
              </v-tooltip>
            </v-slide-x-reverse-transition>
            <v-btn
              color="primary"
              text
              @click="submit"
            >
              Сохранить
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </template>
     <v-snackbar
      v-model="isOperationResultMessage"
      :timeout="messageTimeout"
      :color="operationResultMessageColor"
    >
      {{ operationResultMessage }}
    </v-snackbar>
  </v-app>
</template>

<script>
import axios from 'axios'
import { nextTick } from 'vue';

export default {
  name: 'App',

  data: () => ({
    baseUrl: 'http://37.230.117.222:8090',
    headers: [
      { text: 'Номер заказа', align: 'center', value: 'number' },
      { text: 'Дата заказа', align: 'center', value: 'date' },
      { text: 'Поставщик', align: 'center', value: 'provider.name' },
    ],
    orders: [],
    isGetDialogOpened: false,
    isCreationDialogOpened: false,
    dateFrom: '',
    dateTo: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().slice(0, 10),
    isDateFromOpened: false,
    isDateToOpened: false,
    selectedRow: '',
    selectedProviderName: '',
    selectedOrderDate: '',
    orderItems: [],
    isLoadingActive: false,
    isOrderDateOpened: false,
    providers: [],
    errorMessages: '',
    number: null,
    orderDate: new Date().toISOString().slice(0, 10),
    newOrderTime: new Date(new Date().toISOString()).toString().slice(16, 21),
    provider: null,
    newOrderItems: [ { name: null, quantity: null, unit: null } ],
    isTimePickerOpened: false,
    filters: { numbers: [], itemNames: [], itemUnits: [], providerNames: [] },
    filteredFields: ['numbers', 'itemNames', 'itemUnits', 'providerNames'],
    numbersFilter: null,
    itemNamesFilter: null,
    itemUnitsFilter: null,
    providerNamesFilter: null,
    isOperationResultMessage: false,
    operationResultMessage: '',
    operationResultMessageColor: '',
    messageTimeout: 3000
  }),
  methods: {
    handleRowClick: function(row) {
      // TODO: show popup only when data has been received
      axios.get(this.baseUrl + '/api/v1/order-items', {
        params: {
          orderId: row.id,
        }
      })
        .then(response => {
          if (response.status === 200) {
            this.orderItems = response.data.data;
            this.isGetDialogOpened = true;
            this.selectedRow = row;
            this.selectedProviderName = row.provider.name;
            let currentDate = new Date(row.date).toString();
            this.selectedOrderDate = `${row.date.slice(0, 10)} ${currentDate.slice(16, 21)}`;
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.showResultMessage(response.data.message, 'error');
          }
        })
        .catch(error => {
          this.orderItems = [];
          console.log(error);
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
    },
    getOrders: function() {
      this.isLoadingActive = true;
      axios.get(this.baseUrl + '/api/v1/orders', {
        params: {
          dateFrom: `${this.dateFrom}T00:00:00Z`,
          dateTo: `${this.dateTo}T23:59:59Z`,
          numbersFilter: this.numbersFilter,
          itemNamesFilter: this.itemNamesFilter,
          itemUnitsFilter: this.itemUnitsFilter,
          providerNamesFilter: this.providerNamesFilter
        },
        paramsSerializer: {
          indexes: true
        }
      })
        .then(response => {
          if (response.status === 200) {
            this.orders = response.data.data;
            this.orders.forEach(o => {
              let localDate = new Date(o.date).toString();
              o.date = `${o.date.slice(0, 10)} ${localDate.slice(16, 21)}`;
            });
            this.isLoadingActive = false;
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.orders = [];
            this.showResultMessage(response.data.message, 'error');
          }
        })
        .catch(error => {
          console.log(error);
          this.isLoadingActive = false;
          this.orders = [];
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
    },
    removeOrder: function(id) {
      axios.delete(this.baseUrl + `/api/v1/orders/order/${id}`)
        .then(response => {
          if (response.status === 200) {
            this.isGetDialogOpened = false;
            this.getOrders();
            this.getFilters();
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.showResultMessage(response.data.message, 'error');
          }

        })
        .catch(error => {
          console.log(error);
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
    },
    openCreationPopup: function() {
      axios.get(this.baseUrl + '/api/v1/providers')
        .then(response => {
          if (response.status === 200) {
            this.providers = response.data.data;
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.showResultMessage(response.data.message, 'error');
          }
        })
        .catch(error => {
          console.log(error);
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
      this.isCreationDialogOpened = true;
    },
    resetForm: async function() {
      this.errorMessages = [];
      this.orderDate = new Date().toISOString().slice(0, 10);
      this.newOrderTime = new Date(new Date().toISOString()).toString().slice(16, 21);
      this.newOrderItems = [ { name: null, quantity: null, unit: null } ];
      Object.keys(this.form).forEach(f => {
        this.$refs[f].reset();
      });
      await nextTick();
      Object.keys(this.orderItemsForm).forEach(f => {
        this.$refs[f].forEach(i => {
          i.reset();
        })
      });
    },
    submit: function() {
      let isValidationOk = true;
      Object.keys(this.form).forEach(f => {
        isValidationOk &&= this.$refs[f].validate(true);
      });
      Object.keys(this.orderItemsForm).forEach(f => {
        this.$refs[f].forEach(i => {
          isValidationOk &&= i.validate(true);
        })
      });
      if (isValidationOk) {
        let date = `${this.orderDate} ${this.newOrderTime}:00`;
        date = new Date(date).toISOString();

        if (!this.isGetDialogOpened) {
          axios.post(
            this.baseUrl + '/api/v1/orders/order', 
            {
              number: this.number,
              date: date,
              providerId: this.provider,
              orderItems: this.newOrderItems,
            }
          )
            .then(response => {
              if (response.status === 201) {
                this.isCreationDialogOpened = false;
                this.resetForm();
                this.getOrders();
                this.getFilters();
                this.showResultMessage(response.data.message, 'success');
              } else {
                this.showResultMessage(response.data.message, 'error');
              }
            })
            .catch(error => {
              console.log(error);
              this.showResultMessage(error.response.data.message, 'error');
            })
            .catch(() => {
              this.showResultMessage('Сервер недоступен', 'error');
            });
        } else {
          axios.put(
            this.baseUrl + '/api/v1/orders/order', 
            {
              id: this.selectedRow.id,
              number: this.number,
              date: date,
              providerId: this.provider.id,
              orderItems: this.newOrderItems,
            }
          )
            .then(response => {
              if (response.status === 200) {
                this.isCreationDialogOpened = false;
                this.isGetDialogOpened = false;
                this.resetForm();
                this.getOrders();
                this.getFilters();
                this.showResultMessage(response.data.message, 'success');
              } else {
                this.showResultMessage(response.data.message, 'error');
              }
            })
            .catch(error => {
              console.log(error);
              this.showResultMessage(error.response.data.message, 'error');
            })
            .catch(() => {
              this.showResultMessage('Сервер недоступен', 'error');
            });
        }
      }
    },
    addOrderItem: function() {
      this.newOrderItems.unshift({ name: '', quantity: '', unit: '' });
    },
    deleteOrderItem: function(index) {
      this.newOrderItems.splice(index, 1);
    },
    editOrder: async function() {
      axios.get(this.baseUrl + '/api/v1/providers')
        .then(response => {
          if (response.status === 200) {
            this.providers = response.data.data;
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.showResultMessage(response.data.message, 'error');
          }
        })
        .catch(error => {
          console.log(error);
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
        
      this.isCreationDialogOpened = true;
      await nextTick();
      this.resetForm();
      await nextTick();
      this.number = this.selectedRow.number;
      this.orderDate = this.selectedOrderDate.slice(0, 10);
      this.newOrderTime = this.selectedRow.date.slice(11, 16);
      this.provider = this.selectedRow.provider;
      this.newOrderItems = this.orderItems;
    },
    getFilters: function() {
    axios.get(this.baseUrl + '/api/v1/filters')
        .then(response => {
          if (response.status === 200) {
            this.filters = response.data.data;
            this.showResultMessage(response.data.message, 'success');
          } else {
            this.showResultMessage(response.data.message, 'error');
          }
        })
        .catch(error => {
          console.log(error);
          this.showResultMessage(error.response.data.message, 'error');
        })
        .catch(() => {
          this.showResultMessage('Сервер недоступен', 'error');
        });
    },
    showResultMessage: function(message, status) {
      if (this.isOperationResultMessage) {
        setTimeout(() => {
          this.messageTimeout = 1000;
          this.isOperationResultMessage = true;
          this.operationResultMessage = message;
          this.operationResultMessageColor = status;
        }, 2000);
      } else {
        this.messageTimeout = 3000;
        this.isOperationResultMessage = true;
        this.operationResultMessage = message;
        this.operationResultMessageColor = status;
      }
    }
  },
  computed: {
    form () {
      return {
        number: this.number,
        providerRef: this.provider,
      }
    },
    orderItemsForm() {
      return {
        orderItemNameRef: this.newOrderItems[0].name,
        orderItemUqantityRef: this.newOrderItems[0].quantity,
        orderItemUnitRef: this.newOrderItems[0].unit
      }
    }
  },
  watch: {
    number () {
      this.errorMessages = ''
    },
  },
  mounted() {
    const dateNow = new Date();
    dateNow.setMonth(dateNow.getMonth() - 1);
    this.dateFrom = dateNow.toISOString().slice(0, 10);

    this.getFilters();
  }
};
</script>

<style>
  @import './appStyles.css';
</style>
