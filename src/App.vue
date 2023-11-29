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
    >
    </v-data-table>


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
  }),
  methods: {
    handleClick: function(row) {
      this.isDialogOpened = true;
      console.log(row);
    },
    getOrders: function() {
      axios.get(this.baseUrl + '/api/v1/orders', {
        params: {
          dateFrom: this.dateFrom,
          dateTo: this.dateTo,
        }
      })
      .then(response => {
        if (response.status === 200) {
          this.orders = response.data.data;
        }
      })
      .catch(function(error) {
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
