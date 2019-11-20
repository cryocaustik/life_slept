<template lang="pug">
  .container
    .title How much of your life do you sleep away?
    .card
      .card-header
        .card-header-title Parameters
        .button.is-right.is-info.is-inverted(@click="showHide('parameters')")
          i.fas.fa-eye
      .card-content#parameters(style="display: none")
        form(method="" action="null")
          .columns
            .column
              .field
                label.label Yrs Alive
                .control
                  input.input(type="number" v-model="yrs_alive" @input="hrsSleptGenerator(), chartPercOfYear(), chartTotalPerc()")
              .field
                .control
                  label.checkbox
                    input(type="checkbox" v-model="taking_stuff" @change="hrsSleptGenerator(), chartPercOfYear(), chartTotalPerc()")
                    span(style="margin-left: 0.5em; font-weight: 600;") Taking the Good Stuff?
              .field
                label.label Age Group Map
                  .button.is-info.is-inverted.is-small(@click.stop="showHide('age-map-tbl')")
                    i.fas.fa-eye
                table.table.is-fullwidth.is-hoverable#age-map-tbl(style="display: none;")
                  thead
                    tr
                      td age
                      td hrs slept
                  tbody
                    tr(v-for="yr in age_map")
                      td {{ yr.age }}
                      td
                        input.input.is-small(
                          v-model="yr.hrs"
                          type="number"
                          @input="hrsSleptGenerator(), chartPercOfYear(), chartTotalPerc()"
                        )
            .column
              .field
                label.label Hrs in Year
                .control
                  input.input(v-model="hrs_in_year" disabled)
                .help 
                  pre
                    code (365*24)
              .field
                label.label Hrs Slept Total
                .control
                  input.input(disabled :value="hrsSleptTotal")
                .help
                  pre
                    code sum of (age * hrs per yr) * yrs alive
    .card
      .card-header
        .card-header-title Age Map
        .button.is-right.is-info.is-inverted(@click="showHide('age-map')")
          i.fas.fa-eye
      .card-content(style="display: none;")#age-map
        pre
          code {{ hours_slept }}
    .columns(style="margin-top: 1em;")
      .column.is-one-quarter
        .card
          .card-header
            .card-header-title Percent of Life Slept
            .button.is-inverted.is-info(@click.stop="chartTotalPerc()")
              i.fas.fa-sync-alt
          .card-content
            div#chart-total-perc
      .column
        .card
          .card-header
            .card-header-title Percent of Year Slept
            .button.is-inverted.is-info(@click.stop="chartPercOfYear()")
              i.fas.fa-sync-alt
          .card-content
            div#chart-perc-of-year
</template>
<script>
export default {
  name: "Chart",
  data() {
    return {
      hours_slept: [],
      yrs_alive: 90,
      taking_stuff: false,
      hrs_in_year: 365 * 24,
      age_map: [
        { age: 1, hrs: 12 },
        { age: 4, hrs: 10 },
        { age: 7, hrs: 9 },
        { age: 12, hrs: 8 },
        { age: 19, hrs: 6 },
        { age: 55, hrs: 7 },
        { age: 65, hrs: 8 },
        { age: 75, hrs: 9 }
      ],
      charts: {}
    };
  },
  computed: {
    hrsSleptTotal: function() {
      let hrs_map = this.hours_slept,
        total = [];
      hrs_map.forEach(el => {
        total.push(el.hrs * 365);
      });
      return total.reduce((a, b) => a + b, 0);
    },
    hrsAlive: function() {
      return this.hrs_in_year * this.yrs_alive;
    },
    ageMapDict: function() {
      let age_map = this.age_map,
        age_dict = {};
      age_map.forEach(el => {
        age_dict[el.age] = el.hrs;
      });
      return age_dict;
    }
  },
  methods: {
    showHide: eId => {
      let el = document.getElementById(eId);
      if (el.style.display === "none") {
        el.style.display = "";
      } else if (el.style.display === "" || el.style.display === "") {
        el.style.display = "none";
      }
    },
    hrsSleptGenerator: function() {
      let age_dict = this.ageMapDict;
      let hrs_yr_map = this.age_map;
      let yrs_alive = this.yrs_alive;
      let hrs_slept = [];
      let yr_cnt = 0;
      let hrs;

      hrs_yr_map.forEach(yr => {
        if (yr.age > yrs_alive) {
          return hrs_slept;
        }
        hrs = yr.hrs;
        if (this.taking_stuff === true && yr.age === 19) {
          hrs = 0;
        }
        hrs_slept.push({
          age: yr.age,
          hrs: hrs,
          hrs_yr: hrs * 365,
          hrs_yr_perc: (hrs * 365) / this.hrs_in_year
        });
        yr_cnt = yr.age + 1;
        while (age_dict[yr_cnt] === undefined && yr_cnt < this.yrs_alive) {
          hrs_slept.push({
            age: yr_cnt,
            hrs: hrs,
            hrs_yr: hrs * 365,
            hrs_yr_perc: (hrs * 365) / this.hrs_in_year
          });
          yr_cnt++;
        }
      });
      this.hours_slept = hrs_slept;
      return hrs_slept;
    },
    chartPercOfYear: function() {
      try {
        let data = this.hours_slept;
        if (this.charts.chartPercOfYear) {
          let chart = this.charts.chartPercOfYear;
          chart.load({
            json: data,
            keys: {
              x: "age",
              value: ["hrs_yr_perc"]
            },
            names: {
              age: "Age",
              hrs_yr_perc: "% of Hrs Per Year"
            }
          });
          console.log("chartPercOfYear: data updated");
        } else {
          let chart = c3.generate({
            bindto: "#chart-perc-of-year",
            data: {
              type: "area-spline",
              json: data,
              keys: {
                x: "age",
                value: ["hrs_yr_perc"]
              },
              names: {
                age: "Age",
                hrs_yr_perc: "% of Hrs Per Year"
              }
            },
            axis: {
              y: {
                min: 0,
                padding: { bottom: 0 },
                tick: {
                  format: d3.format("0.0%")
                }
              }
            }
          });
          this.charts["chartPercOfYear"] = chart;
          console.log("chartPercOfYear: new chart generated");
        }
      } catch (error) {
        console.error(`chartPercOfYear: ${error}`);
      }
    },
    chartTotalPerc: function() {
      let data = [
        ["Time Awake", this.hrsAlive],
        ["Time Slept", this.hrsSleptTotal]
      ];
      if (this.charts.chartTotalPerc) {
        let chart = this.charts.chartTotalPerc;
        chart.load({
          columns: data,
          type: "donut"
        });
        console.info("chartTotalPerc: data updated");
      } else {
        let chart = c3.generate({
          bindto: "#chart-total-perc",
          data: {
            columns: data,
            type: "donut"
          }
        });
        this.charts["chartTotalPerc"] = chart;
        console.info("chartTotalPerc: new chart generated");
      }
    }
  },
  mounted() {
    this.hrsSleptGenerator();
    this.chartPercOfYear();
    this.chartTotalPerc();
  }
};
</script>
