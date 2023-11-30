<template>
  <v-app class="table">
    <div>Заказы</div>
    <v-btn
      elevation="2"
      @click="getOrders"
    >
      Получить
    </v-btn>

    <template>
      <v-row>
        <v-col
          cols="6"
        >
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
    </template>

    <v-data-table
      :headers="headers"
      :items="orders"
      :items-per-page="5"
      class="elevation-1 row-pointer"
      @click:row="handleClick"
      :loading="isLoadingActive"
      loading-text="Загружаю. Подождите"
      no-data-text="Нет данных"
    >
    </v-data-table>

    <template>
      <div class="text-center">
        <v-dialog
          v-model="isDialogOpened"
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
                      <td>{{ item.name }}</td>
                      <td>{{ item.quantity }}</td>
                      <td>{{ item.unit }}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
               <v-row class="ma-0">
                  <v-col cols="6" class="d-flex justify-center">
                    <v-btn small color="primary" elevation="5">Редактировать</v-btn>
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
  </v-app>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',

  data: () => ({
    baseUrl: 'http://localhost:5000',
    headers: [
      { text: 'Номер заказа', align: 'center', value: 'number' },
      { text: 'Дата заказа', align: 'center', value: 'date' },
      { text: 'Поставщик', align: 'center', value: 'provider.name' },
    ],
    orders: [],
    isDialogOpened: false,
    dateFrom: '',
    dateTo: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().slice(0, 10),
    isDateFromOpened: false,
    isDateToOpened: false,
    selectedRow: '',
    selectedProviderName: '',
    selectedOrderDate: '',
    orderItems: [],
    isLoadingActive: false,
  }),
  methods: {
    handleClick: function(row) {
      // TODO: show popup when data was received only
      axios.get(this.baseUrl + '/api/v1/order-items', {
        params: {
          orderId: row.id,
        }
      })
      .then(response => {
        if (response.status === 200) {
          this.orderItems = response.data.data;
        }
      })
      .catch(error => {
        this.orderItems = [];
        console.log(error);
      });
      this.isDialogOpened = true;
      this.selectedRow = row;
      this.selectedProviderName = row.provider.name;
      this.selectedOrderDate = row.date.slice(0, 10);
    },
    getOrders: function() {
      this.isLoadingActive = true;
      axios.get(this.baseUrl + '/api/v1/orders', {
        params: {
          dateFrom: this.dateFrom,
          dateTo: this.dateTo,
        }
      })
      .then(response => {
        if (response.status === 200) {
          this.orders = response.data.data;
          this.isLoadingActive = false;
        }
      })
      .catch(error => {
        console.log(error);
        this.isLoadingActive = false;
      });
    },
    removeOrder: function(id) {
      axios.delete(this.baseUrl + `/api/v1/orders/order/${id}`)
      .then(response => {
        if (response.status === 200) {
          // TODO: show message
          this.isDialogOpened = false;
          this.getOrders();
        }
      })
      .catch(error => {
        console.log(error);
      });
    }
  },
  mounted() {
    const dateNow = new Date();
    dateNow.setMonth(dateNow.getMonth() - 1);
    this.dateFrom = dateNow.toISOString().slice(0, 10);
  }
};
</script>

<style>
  @import './appStyles.css';
</style>
