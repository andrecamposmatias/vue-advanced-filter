<template>
  <v-card color="blue-grey darken-1" dark :loading="isUpdating">
    <template v-slot:progress>
      <v-progress-linear
        absolute
        color="green lighten-3"
        height="4"
        indeterminate
      ></v-progress-linear>
    </template>

    <v-form>
      <v-container>
        <v-row>
          <v-col cols="10" align="right">
            <v-chip
              v-for="(filter, index) in advancedFilters"
              v-bind:key="index"
              class="mr-1"              
              close
              @click:close="removeFilterHandler(filter)"
            >
              

              <v-tooltip top>
                <template v-slot:activator="{ on, attrs }">
                  <span v-bind="attrs" v-on="on">
                  <v-avatar left class="mr-0" >
                    <v-icon small :color="filter.color + ' darken-1'">{{ filter.avatar }}</v-icon>
                  </v-avatar>
                  {{ filter.name }}
                  </span>
                </template>
                <span>{{ filter.description }}</span>
              </v-tooltip>

              <v-select
                :items="operators"
                :value="filter.operator"                
                append-icon=""
                hide-details
                class="mx-1 pa-0 ma-0"
                style="width:24px;"                
                single-line
                dense
                v-on:change="filtersOptionsHandler(filter.name, $event, null)"
              >                
              </v-select>

              <v-menu
                v-if="filter.type === 'date'"
                v-model="datePickerMenu[filter.name]"
                :close-on-content-click="false"
                nudge-right="40"
                transition="scale-transition"
                offset-y
                min-width="300px"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field
                    v-show="filter.type === 'date'"
                    :value="formattedDate(filter)"                    
                    readonly
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-show="filter.type === 'date'"
                  v-model="filter.values"
                  range
                  no-title
                  scrollable
                  @input="filtersOptionsHandler(filter.name, null, $event)"
                >
                </v-date-picker> 
                <v-col class="ma-0 white">
                  <v-row class="mx-0 my-2 white" justify="space-around">
                    <v-btn color="primary" small class="px-2" @click="handleDatePicker('TY', filter.name)">
                      <v-icon x-small>far fa-calendar-alt</v-icon>
                      <span class="d-none d-md-flex px-2">This year</span>
                    </v-btn>
                    <v-btn color="primary" small class="px-2" @click="handleDatePicker('TM', filter.name)">
                      <v-icon x-small>far fa-calendar-alt</v-icon>
                      <span class="d-none d-md-flex px-2">This month</span>
                    </v-btn>
                  </v-row>
                  <v-row class="ma-0 white" justify="space-around">
                    <v-btn color="primary" small class="px-2" @click="handleDatePicker('LY', filter.name)">
                      <v-icon x-small>far fa-calendar-alt</v-icon>
                      <span class="d-none d-md-flex px-2">Last year</span>
                    </v-btn>
                    <v-btn color="primary" small class="px-2" @click="handleDatePicker('LM', filter.name)">
                      <v-icon x-small>far fa-calendar-alt</v-icon>
                      <span class="d-none d-md-flex px-2">Last month</span>
                    </v-btn>
                  </v-row>
                </v-col>                              
              </v-menu>

              <v-text-field
                v-show="filter.type === 'text'"
                :style="'width: '+ filter.width || 150 + ';'"
                v-on:change="filtersOptionsHandler(filter.name, null, $event)"
              />

              <v-select
                v-show="filter.type === 'select' || filter.type === 'select-multi'"
                :items="options"
                :multiple="filter.type === 'select-multi'"                
                hide-details                
                class="mx-1 pa-0 ma-0"
                single-line
                dense
                :style="'width: '+ filter.width || 200 + ';'"
                v-on:change="filtersOptionsHandler(filter.name, null, $event)"
              >                
                <template v-slot:selection="{ item:opt, index }">
                  <v-chip v-if="index === 0">
                    <span>{{ opt }}</span>
                  </v-chip>
                  <span
                    v-if="index === 1"
                    class="grey--text text-caption"
                  >
                    (+{{ filter.values.length - 1 }} others)
                  </span>
                </template>
              </v-select>
            </v-chip>
          </v-col>
          <v-col>
            <v-menu offset-y>
              <template v-slot:activator="{ on, attrs }">
                <v-chip color="primary" dark v-bind="attrs" v-on="on">
                  + Add Filter
                </v-chip>
              </template>
              <v-list>
                <v-list-item v-for="(item, index) in filters" v-bind:key="index" :disabled="!!advancedFilters[item.name]">
                  <v-list-item-title v-on:click="!advancedFilters[item.name] && filtersHandler(item)"><v-icon small :color="!!advancedFilters[item.name] ? 'gray' : item.color + ' darken-1'">{{ item.avatar }}</v-icon> {{ item.name }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
            <v-chip
              rounded
              dense
              dark
              color="primary"
              class="ma-2 white--text"
              @click="search = true"
            >
              <v-icon
                left
                dark
              >
                mdi-magnify
              </v-icon>
              Search              
            </v-chip>
          </v-col>
        </v-row>
        <v-row v-if="search">
          <v-col>
            {{ JSON.stringify(advancedFilters) }}
          </v-col>
        </v-row> 
      </v-container>
    </v-form>       
  </v-card>
</template>

<script>
import moment from "moment";
export default {
  name: "AdvanceFilter",
  data() {
    return {
      search: false,
      operators: ["=", "<>"],
      defaultOperator: "=",
      options: ["Alabama", "Alaska", "American Samoa", "Arizona"],
      datePickerMenu: {},
      dateRange: {},
      isUpdating: false,
      filterTitle: "Advance Filter Search",
      advancedFilters: {},
      filters: [
        {
          name: "Date",
          type: "date",
          description: "Select a date or range",
          avatar: 'fa-regular fa-calendar',
          color: 'blue',
          width: '150px'
        },
        {
          name: "Sku",
          type: "text",
          description: "Product SKU#",
          avatar: 'fa-solid fa-box',
          color: 'green',
          width: '110px'
        },
        {
          name: "Status",
          type: "text",
          description: "Return current status",
          avatar: 'fa-solid fa-rotate',
          color: 'green',
          width: '150px'
        },
        {
          name: "Agent",
          type: "select-multi",
          description: "Agent envolved",
          avatar: 'fa-solid fa-user',
          color: 'green',
          width: '250px'
        },
        {
          name: "Reason",
          type: "select",
          description: "Reason the return was made",
          avatar: 'fa-solid fa-right-left',
          color: 'green',
          width: '160px'
        },
      ],
    };
  },

  watch: {
    isUpdating(val) {
      if (val) {
        setTimeout(() => (this.isUpdating = false), 3000);
      }
    },
  },
  computed: {
    
  },
  methods: {
    formattedDate(filter) {
      let initD = filter.values[0];
      let endD = filter.values[1];
      if (endD === undefined) {
        endD = initD;
      }
      if (initD > endD) {
        [initD, endD] = [endD, initD];
      }

      if (initD === endD) {
        return moment(String(initD)).format("DD/MM/YYYY");
      } else {
        return (
          moment(String(initD)).format("DD/MM/YYYY") +
          " - " +
          moment(String(endD)).format("DD/MM/YYYY")
        );
      }
    },
    handleDatePicker(template, filterName) {
      if (template) {
        switch (template) {
          case "TY":
            this.dateRange = [
              moment().startOf("year").format("YYYY-MM-DD"),
              moment().endOf("year").format("YYYY-MM-DD"),
            ];
            break;
          case "TM":
            this.dateRange = [
              moment().startOf("month").format("YYYY-MM-DD"),
              moment().endOf("month").format("YYYY-MM-DD"),
            ];
            break;
          case "LY":
            this.dateRange = [
              moment().subtract(1, "year").startOf("year").format("YYYY-MM-DD"),
              moment().subtract(1, "year").endOf("year").format("YYYY-MM-DD"),
            ];
            break;
          case "LM":
            this.dateRange = [
              moment()
                .subtract(1, "month")
                .startOf("month")
                .format("YYYY-MM-DD"),
              moment().subtract(1, "month").endOf("month").format("YYYY-MM-DD"),
            ];
            break;
          default:
            console.log("Option invalid");
        }
      }
      
      if (this.dateRange && this.dateRange.length >= 2) {
        this.datePickerMenu[filterName] = false;
        this.filtersOptionsHandler(filterName, null, this.dateRange);
      }
    },
    filtersOptionsHandler(filterName, operator, values) {
      console.log("filtersOptionsHandler called", filterName);

      if (operator) this.advancedFilters[filterName].operator = operator;
      if (values) this.advancedFilters[filterName].values = values;
      
      console.log("updated filter", this.advancedFilters);
      this.$forceUpdate();
    },
    filtersHandler(newFilter) {
      console.log("filtersHandler called");
      this.advancedFilters[newFilter.name] = {
        name: newFilter.name,
        type: newFilter.type,
        description: newFilter.description,
        avatar: newFilter.avatar,
        color: newFilter.color,
        width: newFilter.width,
        operator: this.defaultOperator,
        values: newFilter.type === 'date' ? [moment().format("YYYY-MM-DD")] : null,
        //values: newFilter.type === 'date' ? [moment().format("YYYY-MM-DD"), moment().format("YYYY-MM-DD")] : null,
      };      
      this.$forceUpdate();
      console.log("added filter", this.advancedFilters);
    },
    removeFilterHandler(item) {
      console.log("removeFilterHandler called");
      if (item.name) {
        delete this.advancedFilters[item.name];
      }
      console.log("advancedFilters", this.advancedFilters);
      this.$forceUpdate();
    },
  },
};
</script>

<style>
.v-select .v-select__selections input {
    display: none !important;;
}
</style>
