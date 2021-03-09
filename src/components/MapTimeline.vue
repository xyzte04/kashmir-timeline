<template>
  <div id="map-timeline">
    <div id="date-dist-prominent">
      <div id="date-prominent">Date: <span></span></div>
      <div id="dist-prominent">District: <span></span></div>
    </div>
    <div id="map">
      <div id="hover-note">
        Hover over the map <span class="last-show">and dots</span> to explore.
        <br /><span class="last-show"
          >Click on a district or dot to see cases</span
        >
      </div>

      <div id="tooltip">
        <div id="date">Date: <span></span></div>
        <div id="dist">District: <span></span></div>
        <div>Civilians Killed: <span id="civ_killed"></span></div>
        <div>Civilians Injured: <span id="civ_injured"></span></div>
        <!-- <div>Injured: <span id="injured_names"></span></div>
        <div>Killed: <span id="injured_names"></span></div> -->
      </div>
    </div>
  </div>
  <div id="timeline"></div>
</template>

<script>
import * as d3 from "d3";
import { bigR, defaultR } from "../js/constants";

function clean(s) {
  return s
    .trim()
    .replace(/[()]/gi, "")
    .replace(/ /gi, "-")
    .toLowerCase();
}

function toTitleCase(str) {
  return str.replace(/\w\S*/g, function(txt) {
    return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
  });
}

var tp = d3.timeParse("%d-%b-%y");
var tf = d3.timeFormat("%b");
var tf2 = d3.timeFormat("%d %B %Y");

var centroids = {};
var projection;

window.disableMapClick = true;

var mobile = window.innerWidth <= 500;

d3.json("data/jnk.json").then(mapData => {
  d3.json("data/jnk-boundaries.json").then(mapDataBoundaries => {
    d3.csv("data/tii-tca.csv").then(data => {
      mapData.features.forEach(f => {
        f.properties.dist_name = clean(
          f.properties.DISTRICT.toLowerCase() === "shopiyan"
            ? "shupiyan"
            : f.properties.DISTRICT.toLowerCase() === "bandipura"
            ? "bandipore"
            : f.properties.DISTRICT.toLowerCase() === "badgam"
            ? "budgam"
            : f.properties.DISTRICT.toLowerCase() === "baramula"
            ? "baramulla"
            : f.properties.DISTRICT.toLowerCase()
        );
      });

      // console.log(mapDataBoundaries);
      var w = window.innerWidth >= 500 ? 500 : window.innerWidth * 0.98;
      var h = (w * 3) / 5;

      window.mapWidth = w;
      window.mapHeight = h;

      projection = d3
        .geoMercator()
        .scale(1300)
        .translate([75.3412, 33.2778])
        .fitSize([w, h], mapData);

      window.geoPath = d3.geoPath(projection);

      var svg = d3
        .select("#map")
        .append("svg")
        .attr("class", "map-container")
        .attr("width", w)
        .attr("height", h)
        .append("g")
        .attr("class", "map-container-g");

      var mapHoverLabel = svg.append("g").attr("class", "map-hover-label");

      var mapLabel = svg.append("g").attr("class", "map-label");

      mapHoverLabel.append("text").attr("class", "map-hover-label-stroke");
      mapHoverLabel.append("text").attr("class", "map-hover-label-text");

      mapLabel.append("text").attr("class", "map-label-stroke");
      mapLabel.append("text").attr("class", "map-label-text");

      function handleMouseOver() {
        var xy = projection(
          centroids[clean(this.__data__.properties.dist_name)]
        );
        xy[0] = xy[0] - 10;
        xy[1] = xy[1] - 10;

        var nm = this.__data__.properties.DISTRICT;

        if (nm === "Leh (Ladakh)") nm = "Leh";

        mapHoverLabel
          .attr("transform", `translate(${xy[0]}, ${xy[1]})`)
          .style("opacity", 1)
          .selectAll("text")
          .text(nm);

        mapHoverLabel.raise();

        d3.select(this).classed("map-hover-highlight", true);

        d3.select(".jnk-label")
          .transition()
          .duration(200)
          .style("opacity", 0);
      }

      function handleClickMap() {
        if (window.disableMapClick) return;

        var xy = projection(
          centroids[clean(this.__data__.properties.dist_name)]
        );
        xy[0] = xy[0] - 10;
        xy[1] = xy[1] - 10;

        var circleClass = ".case-circle-" + this.__data__.properties.dist_name;

        d3.selectAll(".map-district").classed("map-click-highlight", false);
        d3.selectAll(".map-district").classed("map-hover-highlight", false);
        d3.selectAll(".map-district").classed("map-highlight", false);

        d3.select(this).classed("map-click-highlight", true);

        d3.selectAll("circle")
          .transition()
          .attr("r", defaultR);

        d3.selectAll(circleClass)
          .transition()
          .attr("r", bigR);

        d3.select("#tooltip").style("opacity", 0);
      }

      function handleMouseOut() {
        d3.select(this).classed("map-hover-highlight", false);

        mapHoverLabel.style("opacity", 0);

        d3.select(".jnk-label")
          .transition()
          .duration(200)
          .style("opacity", 1);
      }

      svg
        .selectAll("path")
        .data(mapData.features)
        .enter()
        .append("path")
        .attr("class", d => {
          centroids[clean(d.properties.dist_name)] = d3.geoCentroid(d);
          return clean(d.properties.dist_name);
        })
        .classed("map-district", true)
        .attr("d", window.geoPath)
        .on("mouseover", handleMouseOver)
        .on("touchmove", handleMouseOver)
        .on("click", handleClickMap)
        .on("mouseout", handleMouseOut);

      svg
        .selectAll("path.map-district-boundaries")
        .data(mapDataBoundaries.features)
        .enter()
        .append("path")
        // .attr("class", d => {
        //   centroids[clean(d.properties.DISTRICT)] = d3.geoCentroid(d);
        //   return clean(d.properties.DISTRICT) + "-boundaries";
        // })
        .classed("map-district-boundaries", true)
        .attr("d", window.geoPath);

      var jnkLabel = svg
        .append("g")
        .attr("class", "jnk-label")
        .classed("dark-greyed", true);

      jnkLabel
        .append("text")
        .attr("class", "jnk-label-border")
        .attr("y", h / 2 + 100)
        .attr("x", w / 2 - 90)
        .text("Jammu");

      jnkLabel
        .append("text")
        .attr("class", "jnk-label-border")
        .attr("y", h / 2 + 20)
        .attr("x", w / 2 + 30)
        .text("Leh");

      jnkLabel
        .append("text")
        .attr("class", "jnk-label-border")
        .attr("x", w / 2 - 100)
        .attr("y", h / 2 + 50)
        .text("Kashmir");

      jnkLabel
        .append("text")
        .attr("y", h / 2 + 100)
        .attr("x", w / 2 - 90)
        .text("Jammu");

      jnkLabel
        .append("text")
        .attr("y", h / 2 + 20)
        .attr("x", w / 2 + 30)
        .text("Leh");

      jnkLabel
        .append("text")
        .attr("x", w / 2 - 100)
        .attr("y", h / 2 + 50)
        .text("Kashmir");

      // .attr("text-anchor", "middle")
      // .attr("y", h / 2 + 20)
      // .html(
      //   `<tspan x=${w / 2 + 30}>Jammu</tspan><tspan dy=25 x=${w / 2 +
      //     30}> and Kashmir</tspan>`
      // );

      // jnkLabel;

      svg.append("text").attr("class", "jnk-label");

      makeTimeline(data);
    });
  });
});

function makeTimeline(timelineData) {
  timelineData = timelineData.map(d => ({
    civ_killed: +d.CIV,
    civ_injured: +d.CIV2,
    civ_killed_names: d.KILLED_NAME === "" ? +d.CIV : d.KILLED_NAME,
    civ_injured_names: d.INJURED_NAME === "" ? +d.CIV2 : d.INJURED_NAME,
    dist:
      d.DIST.toLowerCase() === "shopiyan"
        ? "shupiyan"
        : d.DIST.toLowerCase() === "bandipura"
        ? "bandipore"
        : d.DIST.toLowerCase() === "badgam"
        ? "budgam"
        : d.DIST.toLowerCase(),
    dist_name: d.DIST,
    date: tp(d.DATE),
    date_str: tf2(tp(d.DATE))
  }));

  var w = window.innerWidth >= 1200 ? 1200 : window.innerWidth * 0.98;
  var h = 200;

  var m = {
    l: 20,
    t: 20,
    r: 20,
    b: 20
  };

  var timelineSvg = d3
    .select("#timeline")
    .style("top", window.innerHeight - (mobile ? 50 : h) - 100 + "px")
    .append("svg")
    .attr("class", "timeline-container")
    .attr("width", w)
    .attr("height", h)
    .append("g")
    .attr("transform", `translate(${m.l}, ${m.t})`);

  w = w - m.l - m.r;
  h = h - m.t - m.b;

  var x = d3
    .scaleTime()
    .range([0, w])
    .domain([tp("01-Jan-20"), tp("31-Dec-20")]);

  var y = d3
    .scaleBand()
    .range([h, 0])
    .domain(d3.range(0, 6));

  var nest = Array.from(
    d3.group(timelineData, d => d.date_str),
    ([key, values]) => ({ key, values })
  );

  var c = 0;
  // var t = 0;

  function handleMoveCircle() {
    d3.selectAll("circle")
      .classed("circle-highlight", false)
      .attr("r", 5);

    d3.selectAll(".map-district").classed("map-highlight", false);
    d3.selectAll(".map-district").classed("map-hover-highlight", false);
    d3.selectAll(".map-district").classed("map-click-highlight", false);

    d3.select(this)
      .classed("circle-highlight", true)
      .attr("r", 10);
    d3.select("." + this.__data__.dist).classed("map-highlight", true);
    d3.select("#tooltip #date > span").text(this.__data__.date_str);
    d3.select("#tooltip #dist > span").text(
      toTitleCase(this.__data__.dist_name)
    );
    d3.select("#tooltip #civ_injured").text(this.__data__.civ_injured_names);
    d3.select("#tooltip #civ_killed").text(this.__data__.civ_killed_names);

    d3.select(this).classed("map-hover-highlight", true);

    d3.select("#tooltip")
      .style("opacity", 1)
      .style("display", "block");
  }

  nest.forEach(date => {
    date.values.forEach((attack, i) => {
      timelineSvg
        .append("circle")
        .datum(attack)
        .attr("r", 5)
        .attr("cx", x(attack.date))
        // .attr("cy", h + 50)
        .attr("cy", y(i))
        .classed("civ-killed", attack.civ_killed > 0)
        .classed(
          "civ-injured",
          attack.civ_injured > 0 && attack.civ_killed === 0
        )
        .classed(
          "attack-simple",
          attack.civ_injured === 0 && attack.civ_killed === 0
        )
        .classed("spl-circle", attack.civ_killed > 0 || attack.civ_injured)
        .classed("subtle", true)
        .classed(`case-circle-${attack.dist}`, true)
        .attr("data-date", date.key)
        .classed(`data-index-${c++}`, true)
        .attr("data-district", attack.dist)
        .on("mouseover", handleMoveCircle)
        .on("touchmove", () => {
          console.log("touchmove");
        });
      // .on("mouseout", function() {
      //   d3.select(this)
      //     .classed("circle-highlight", false)
      //     .attr("r", 5);
      //   d3.select("." + this.__data__.dist).classed("map-highlight", false);
      //   // d3.select("#tooltip").style("opacity", 0);
      // });

      d3.selectAll(".spl-circle").raise();
      // .transition()
      // .duration(t)
      // .attr("cy", y(i));

      // t += 10;
    });
  });

  timelineSvg
    .append("g")
    .attr("transform", `translate(${0}, ${h})`)
    .call(d3.axisBottom(x).tickFormat(tf));
}

export default {
  name: "MapTimeline"
};

export { centroids, projection };
</script>

<style lang="scss">
$red: rgb(252, 99, 99);
$red-light: rgb(255, 162, 162);
$orange: rgb(252, 196, 99);
$orange-dark: rgb(194, 143, 56);
#map-timeline {
  position: sticky;
  top: 100px;
  // left: 50%;
  // transform: translateX(-50%);
  z-index: 100;
  opacity: 0;

  svg {
    overflow: hidden;
  }

  #map {
    text-align: left;
    padding-left: 200px;
    position: relative;

    #hover-note {
      position: absolute;
      bottom: -30px;
      left: 30px;
      padding-left: 200px;
      color: white;
      font-size: 0.8rem;
    }
    .map-container {
      path {
        stroke: black;
        stroke-width: 0.2px;
        fill: #ccc;

        &.map-highlight,
        &.map-hover-highlight,
        &.map-click-highlight {
          fill: $red !important;
        }
      }

      .map-district.data-not-available {
        fill: none;
      }

      .map-district-boundaries {
        pointer-events: none;
        fill: none;
        stroke: white;
        stroke-opacity: 1;
        stroke-width: 3px;
      }
      &.hide-jammu {
        path.kargil,
        path.leh,
        path.jammu,
        path.doda,
        path.kathua,
        path.ramban,
        path.reasi,
        path.kishtwar,
        path.punch,
        path.rajouri,
        path.udhampur,
        path.samba {
          fill: #4d4d4d;
        }
      }
    }

    .jnk-label {
      pointer-events: none;
      font-size: 1.2rem;
      &.dark-greyed {
        text {
          fill: #a87373;
          &.jnk-label-border {
            stroke-width: 2px;
            stroke: white;
          }
        }
      }
      text {
        fill: white;
        &.jnk-label-border {
          stroke-width: 2px;
          stroke: black;
        }
      }
    }

    // .jnk-label-border {
    //   pointer-events: none;
    //   font-size: 1.2rem;
    //   &.dark-greyed {
    //     text {
    //       stroke: #a87373;
    //       stroke-width: 4px;
    //     }
    //   }
    //   text {
    //     stroke: white;
    //     stroke-width: 4px;
    //   }
    // }

    .map-hover-label,
    .map-label {
      pointer-events: none;
      text {
        font-size: 1rem;
        fill: #dcd9d9;
        font-weight: bold;
      }
      .map-hover-label-stroke,
      .map-label-stroke {
        stroke: #333;
        stroke-width: 2px;
        stroke-linecap: round;
      }
    }
  }

  text {
    fill: white;
  }

  #tooltip {
    display: none;
    opacity: 0;
    pointer-events: none;
    position: absolute;
    left: 35%;
    padding: 10px;
    border-radius: 2px;
    background-color: #ffffffdd;
    border: 1px solid #ddd;

    div {
      span {
        // font-weight: bold;
        font-weight: bold;
        &#civ_killed {
          color: $red;
        }

        &#civ_injured {
          color: $orange-dark;
        }
      }
    }
  }
}

#date-dist-prominent {
  position: absolute;
  padding: 10px;
  padding-left: 200px;
  border-radius: 0 5px 5px 0;
  font-size: 1.1rem;
  color: #ccc;
  font-weight: bold;
  background-color: #3a3a3a;
  // opacity: 0.4 !important;
  z-index: 10;
  opacity: 0;

  span {
    color: $red-light;
  }
}

#timeline {
  position: sticky;
  // left: 50%;
  // transform: translateX(-50%);
  z-index: -1;
  opacity: 0;
  text-align: center;
  opacity: 0;

  svg {
    overflow: visible;
  }

  .tick > line,
  .domain {
    stroke: white;
  }

  .tick > line {
    display: none;
  }

  .tick > text {
    font-size: 15px;
    color: white;
  }

  circle {
    fill: #ccc;
    stroke: gray;
    fill-opacity: 0.7;
    &.civ-injured {
      fill: $orange;
      fill-opacity: 0.7;
      stroke: $orange;
    }
    &.civ-killed {
      fill: $red;
      fill-opacity: 0.7;
      stroke: $red;
    }
    &.subtle {
      // fill: #ccc;
      // stroke: gray;
    }
  }
}

@media (max-width: 500px) {
  #map-timeline {
    position: sticky;
    top: 300px;
    z-index: -100;
    #map {
      padding-left: 0px;
      #hover-note {
        padding-left: 0px;
      }
    }
  }
  #date-dist-prominent {
    padding-left: 10px;
  }
}
</style>
