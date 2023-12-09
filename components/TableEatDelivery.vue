
<template>
  <v-card
    flat
    title="Nutrition"
  >
    <span :style="{backgroundColor: '#ECD9D9'}">{{ new Date() }}</span>
    <v-btn @click="customSort1()">1</v-btn>
    <v-btn @click="customSort2()">2</v-btn>

    <v-select
      label="Filter Status Subscribes"
      :items="filterStatus"
      item-title="key"
      item-value="sort_terms"
      item-text="key"
      v-model="search"
    ></v-select>

    <v-span v-model="search" :style="{backgroundColor: '#ECD9D9'}">Данные Селекта: {{ typeof search }}</v-span>

    <template v-slot:text>
      <v-text-field
        v-model="search"
        label="Search"
        prepend-inner-icon="mdi-magnify"
        single-line
        variant="outlined"
        hide-details
      ></v-text-field>
    </template>

    <v-data-table
      :headers="headers"
      :items="diets"
      :search="search"
      :custom-filter="customFilter"
    >
      <template v-slot:item.diets = "{ item }">
        <tr v-for="subitem in item.diets" class="elevation-1">
          {{subitem}}
        </tr>
      </template>

      <template v-slot:item.tariff = "{ item }">
        <tr v-for="subitem in item.tariff" class="elevation-1">
          {{subitem}}
        </tr>
      </template>

      <template v-slot:item.pay_status = "{ item }">
        <td 
          :style="{backgroundColor: (item.order_payed !== '0.00' && item.order_sum > 0 ? '#ECD9D9' : (item.order_payed === '0.00' && item.order_sum > 0 ? '#D9E2EC' : '#D4ECC9') ) }"
        >
          <tr class="elevation-1"> {{ item.order_sum }} </tr>
          <tr class="elevation-1"> {{ item.pay_status }} </tr>
          <tr class="elevation-1"> {{ item.order_payed }} </tr>
        </td>
      </template>

      <template v-slot:item.dates = "{ item }">
        <tr v-for="subitem in item.dates" class="elevation-1">
          {{subitem.start_date}} / {{ subitem.end_date }}
        </tr>
      </template>

      <template v-slot:item.quantity_dates = "{ item }">
        <tr v-if="item.sort_terms===1" v-for="subitem in item.quantity_dates" class="elevation-1">
          Закончилась {{subitem}} дней
        </tr>
        <tr v-else-if="item.sort_terms===5" v-for="subitem in item.quantity_dates" class="elevation-1">
          Заканчивается через {{subitem}} дней
        </tr>
        <tr v-else-if="item.sort_terms===7" v-for="subitem in item.quantity_dates" class="elevation-1">
          Начинается через {{subitem}} дней
        </tr>
        <tr v-else="item.sort_terms===2||3||4||6" v-for="subitem in item.quantity_dates" class="elevation-1">
          {{subitem}}
        </tr>
      </template>
    
    </v-data-table>
  </v-card>
</template>

<script>
import json from '../data/data.json'
  export default {
    data () {
      return {
        search: '',
        select:'',
        headers: [
          {
            align: 'start',
            key: 'o_id',
            sortable: false,
            title: ' ID ',
          },
          { key: 'client_name', title: 'Clients Name', sortable: false },
          { key: 'diets', title: 'Diets Names', sortable: false },
          { key: 'tariff', title: 'Tariff', sortable: false },
          { key: 'address', title: 'Address', sortable: false },
          { key: 'phone', title: 'Contacts', sortable: false },
          { key: 'pay_status', title: 'Pay Status', sortable: false },
          { key: 'courier_comment', title: 'Courier Notes', sortable: false },
          { key: 'inner_comment', title: 'Admin Notes', sortable: false },
          { key: 'dates', title: 'Dates Subscribe', sortable: false },
          { key: 'quantity_dates', title: 'Quantity Days', sortable: false },
        ],
        diets: [],
        arr1: [], 
        arr2: [], 
        arr3: [], 
        arr4: [], 
        arr5: [], 
        arr6: [], 
        arr7: [],

        filterStatus: [ 
                        {key: "Завершенные", sort_terms: 1}, 
                        {key: "Заканчиваются сегодня", sort_terms: 2 || 3},
                        {key: "Заканчиваются завтра", sort_terms: 4},
                        {key: "Заканчиваются через: ", sort_terms: 5},
                        {key: "Начинаются сегодня", sort_terms: 6},
                        {key: "Начинаются через: ", sort_terms: 7},
                      ]
      }
    },

    created() { 
     let nowTime = new Date();
    // let nowTime = new Date(2023, 12, 4);
    // let nowDate = nowTime.getDate(); 
      for (let j=0; j<json.data.length; j++){
        json.data[j].quantity_dates = [];
        json.data[j].sort_terms = null;
        for (let i=0; i<json.data[j].dates.length; i++) {
          let startTime = new Date(json.data[j].dates[i].start_date);
         // let startDate = startTime.getDate()
          let endTime = new Date(json.data[j].dates[i].end_date);
         // let endDate = endTime.getDate()
         // console.log(startTime, endTime, nowTime);
          if(endTime < nowTime){
            let diff = nowTime.getTime() - endTime.getTime();
            let terms = Math.floor(diff / (1000 * 60 * 60 * 24));
            if(terms === 0){
              json.data[j].quantity_dates.push("Заканчивается сегодня"); // "Заканчивается сегодня"
              json.data[j].sort_terms = 2;
            } else {
              json.data[j].quantity_dates.push( terms ); // "Закончилось " + terms + " дней назад"
              json.data[j].sort_terms = 1;
            }
          } else if(startTime < nowTime && endTime > nowTime) {
            let diff = endTime.getTime() - nowTime.getTime();
            let terms = Math.floor(diff / (1000 * 60 * 60 * 24));
            if(terms === 0){
              json.data[j].quantity_dates.push("Заканчивается сегодня"); // "Заканчивается сегодня"
              json.data[j].sort_terms = 3;
            } else if(terms === 1){
              json.data[j].quantity_dates.push("Заканчивается завтра"); // "Заканчивается завтра"
              json.data[j].sort_terms = 4;
            } else if(terms > 1){
              json.data[j].quantity_dates.push( terms ); // "Заканчивается через " + terms + " дней"
              json.data[j].sort_terms = 5;
            } 
          }
          else if(startTime > nowTime){
            let diff = startTime.getTime() - nowTime.getTime();
            let terms = Math.floor(diff / (1000 * 60 * 60 * 24));
            if(terms === 0){
              json.data[j].quantity_dates.push("Начинается сегодня"); // "Начинается сегодня"
              json.data[j].sort_terms = 6;
            } else {
              json.data[j].quantity_dates.push( terms ); // "Начинается через " + terms + " дней"
              json.data[j].sort_terms = 7;
            }
          }
        }
      }
      this.diets = json.data;
      this.sortForTerms();
    },

    watch: {

    },

    computed: {
//      currentItems(){
//        return this.diets.filter(val=>val.sort_terms === parseInt(this.search))
//      },
    },

    props: {
//      customFilter(){
//        return items.filter(el => filter(el.sort_terms === search))
//      },
    },

    methods: {

    customFilter(items, search, filter) {
//      search = search.toString().toLowerCase()
      return items.filter(row => filter(row["sort_terms"], search));
    },

//    customFilter(){     
//      return items.filter(el => filter(el.sort_terms === search))
//    },
    
    customSort1(){
        this.arr1.sort((a, b) => Math.min.apply(Math, b.quantity_dates) - Math.min.apply(Math, a.quantity_dates));
        // this.arr1.map()
        this.arr5.sort((a, b) => Math.min.apply(Math, a.quantity_dates) - Math.min.apply(Math, b.quantity_dates));
        this.arr7.sort((a, b) => Math.min.apply(Math, a.quantity_dates) - Math.min.apply(Math, b.quantity_dates));

        this.diets = [].concat( this.arr1, this.arr2, this.arr3, this.arr4, this.arr5, this.arr6, this.arr7 ); 
      },

      customSort2(){
        this.arr1.sort((a, b) => Math.min.apply(Math, a.quantity_dates) - Math.min.apply(Math, b.quantity_dates));
        // this.arr1.map()
        this.arr5.sort((a, b) => Math.min.apply(Math, a.quantity_dates) - Math.min.apply(Math, b.quantity_dates));
        this.arr7.sort((a, b) => Math.min.apply(Math, b.quantity_dates) - Math.min.apply(Math, a.quantity_dates));

        this.diets = [].concat( this.arr7, this.arr6, this.arr5, this.arr4, this.arr3, this.arr2, this.arr1 ); 
      },
      sortForTerms() {
        for (let j=0; j<this.diets.length; j++){
          if( this.diets[j].sort_terms === 1){           
            this.arr1.push(this.diets[j]);
          } else if (this.diets[j].sort_terms === 2){
            this.arr2.push(this.diets[j])
          } else if (this.diets[j].sort_terms === 3){
            this.arr3.push(this.diets[j])
          } else if (this.diets[j].sort_terms === 4){
            this.arr4.push(this.diets[j])
          } else if (this.diets[j].sort_terms === 5){
            this.arr5.push(this.diets[j])
          } else if (this.diets[j].sort_terms === 6){
            this.arr6.push(this.diets[j])
          } else if (this.diets[j].sort_terms === 7){
            this.arr7.push(this.diets[j])
          };
        };
      }
    }
  }
</script>
