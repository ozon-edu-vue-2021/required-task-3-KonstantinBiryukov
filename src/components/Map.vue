<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div
        v-if="!isLoading"
        class="map-root"
    >
      <MapSVG ref="svg" v-click-outside="hide"/>
      <TableSVG ref="table"></TableSVG>
    </div>
    <div v-if="!isLoading">
      <PersonCard v-if="personCardShow" :person="person"></PersonCard>
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import MapSVG from "@/assets/images/map.svg";
import TableSVG from "@/assets/images/workPlace.svg";
import PersonCard from "./SideMenu/PersonCard";
import ClickOutside from 'vue-click-outside'

import * as d3 from "d3";
import tables from "@/assets/data/tables.json"
import legend from "@/assets/data/legend.json";
import people from "@/assets/data/people.json";

export default {
  name: "Map",
  components: {
    MapSVG, TableSVG, PersonCard
  },
  directives: {
    ClickOutside
  },
  data() {
    return {
      isLoading: false,
      svg: null,
      g: null,
      tables: [],
      tableSVG: null,
      people: [],
      personCardShow: false,
      person: null
    };
  },
  mounted() {
    try {
      this.svg = d3.select(this.$refs.svg);
      this.g = this.svg.select("g");
      this.tables = tables;
      this.tableSVG = d3.select(this.$refs.table);
      this.drawTables();

      this.people = people;
      this.initListeners();
    } catch (e) {
      console.log(e)
    }
  },
  methods: {
    initListeners() {
      const tables = document.body.querySelectorAll(".employer-place");
      tables.forEach(table => {
        table.addEventListener("click", () => {
          const _id = parseInt(table.firstElementChild.getAttribute("_id"));
          const person = people.find(person => person._id === _id);
          if (person) {
            this.person = person;
            this.personCardShow = true;
          }
        })
      })
    },
    hide() {
      this.personCardShow = false;
    },
    drawTables() {
      // Создаем группу для рабочих мест внутри SVG карты (внутри элемента "g");
      const svgTablesGroup = this.g.append("g").classed("groupPlaces", true);

      // каждый стол должен быть добавлен в нашу группу рабочих мест
      this.tables.forEach(table => {
        // создать группу для рабочего стола (добавим еще одну группу "g" в svgTableGroup),
        // в этой группе мы расположим нужную координату, передвигая стол на координату x,y
        const svgTable = svgTablesGroup
            .append("g")
            .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
            .attr("id", table.id)
            .classed("employer-place", true);

        // добавим сами столы с поворотом, создаем еще одну группу "g", на этот раз внутри svgTable.
        // а затем покрасим столы, информация о цвете находится в /legend.json с п привязкой по подразделениям (group_id)
        svgTable
            .append("g")
            .attr("transform", `rotate(${table.rotate || 0})`)
            .attr("group_id", table.group_id)
            .attr("_id", table._id)
            .html(this.tableSVG.html())
            .attr( // устанавливаем цвет подразделения
                "fill",
                legend.find(it => it.group_id === table.group_id)
                    ?.color ?? "transparent"
            );
      });
    }
  }
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
