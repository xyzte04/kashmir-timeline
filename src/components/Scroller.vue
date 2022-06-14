<template>
  <div id="scroll-container">
    <div
      v-bind:class="
        'scroll-section' + ' ' + (item.fullScreen ? 'full-screen' : '')
      "
      v-for="item in items"
      :key="item.message"
      v-bind:data-blankSpace="item.blankSpace"
    >
      <div
        class="scrolly-text-container"
        v-bind:class="
          item.className + ' ' + (item.fullScreen ? 'full-screen' : '')
        "
        v-bind:data-index="item.index"
        v-bind:data-fullScreen="item.fullScreen"
        v-bind:data-className="item.className"
        v-bind:data-color="item.outerClassName"
        v-bind:data-blankSpace="item.blankSpace"
      >
        <div class="scrolly-text" v-html="item.message"></div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
require("waypoints/lib/noframework.waypoints.js");
import * as d3 from "d3";
import { defaultR, bigR } from "../js/constants";
// import { centroids } from "./MapTimeline";

var colors = {
  center: "#3a3a3a",
  "hide-viz": "#3a3a3a",
  "highlight-kashmir": "#3a3a3a",
  democratic: "#3a3a3a",
  "all-dots": "#3a3a3a",
  "injured-dots": "#3a3a3a",
  "killed-dots": "#3a3a3a",
  "greyed-dots": "#3a3a3a",
  // "story-starts": d3.schemeCategory10[0],
  "story-starts": "#3a3a3a",
  siraj: "#BA7D5A",
  pandita: "#385A69",
  "waseem-bari": "#864B36",
  khanday: "#51494C",
  najar: d3.schemeCategory10[5],
  "babar-qadri": "#385A69",
  altaf: "#51494C",
  ykpora: "#385A69",
  last: "#3a3a3a"
};

var jammuDistricts = [
  "path.kargil",
  "path.leh",
  "path.jammu",
  "path.doda",
  "path.kathua",
  "path.ramban",
  "path.reasi",
  "path.kishtwar",
  "path.poonch",
  "path.rajouri",
  "path.udhampur",
  "path.samba"
];

var kashmirDistricts = [
  "path.anantnag",
  "path.budgam",
  "path.bandipore",
  "path.baramulla",
  "path.ganderbal",
  "path.kulgam",
  "path.kupwara",
  "path.pulwama",
  "path.shupiyan",
  "path.srinagar"
];

function isLetter(str) {
  return str.length === 1 && str.match(/[a-z]/i);
}

// var geoPath = window.geoPath;

// var svg = d3.select(".map-container");
var g = d3.select(".map-container-g");

// function zoomed(event) {
//   console.log(g)
//   const { transform } = event;
//   g.attr("transform", transform);
//   g.attr("stroke-width", 1 / transform.k);
// }

// const zoom = d3
//   .zoom()
//   .scaleExtent([1, 8])
//   .on("zoom", zoomed);

// svg.call(zoom);

// function mapZoom(path) {
//   return;
//   const [[x0, y0], [x1, y1]] = window.geoPath.bounds(path.__data__);

//   svg
//     .transition()
//     .duration(50)
//     .call(
//       zoom.transform,
//       d3.zoomIdentity
//         .translate(window.mapWidth / 2, window.mapHeight / 2)
//         .scale(
//           Math.min(
//             8,
//             0.9 /
//               Math.max(
//                 (x1 - x0) / window.mapWidth,
//                 (y1 - y0) / window.mapHeight
//               )
//           )
//         )
//         .translate(-(x0 + x1) / 2, -(y0 + y1) / 2)
//     );
// }

window.currentZoom = null;

function mapZoom(path) {
  return;
  var x, y, k;
  var d = path.__data__;

  if (d.properties.dist_name === window.currentZoom) return;
  else {
    d.properties.dist_name = window.currentZoom;
  }

  if (d) {
    var centroid = window.geoPath.centroid(d);
    x = centroid[0];
    y = centroid[1];
    k = 4;
    // centered = d;
  } else {
    x = window.mapWidth / 2;
    y = window.mapHeight / 2;
    k = 1;
    // centered = null;
  }

  // g.selectAll("path")
  //     .classed("active", centered && function(d) { return d === centered; });

  // console.log("translate(" + window.mapWidth / 2 + "," + window.mapHeight / 2 + ")scale(" + k + ")translate(" + -x + "," + -y + ")")

  setTimeout(() => {
    g.attr(
      "transform",
      "translate(" +
        window.mapWidth / 2 +
        "," +
        window.mapHeight / 2 +
        ")scale(" +
        k +
        ")translate(" +
        -x +
        "," +
        -y +
        ")"
    );
  }, 0);
}

function highlight(el, self, log = false) {
  if (log) {
    console.log(el, self);
  }
  d3.select("#timeline").style("z-index", -1);
  d3.selectAll(".map-district").classed("map-highlight", false);
  // d3.select("." + el.__data__.dist).classed("map-highlight", true);

  d3.selectAll("circle")
    .classed("subtle", true)
    .transition()
    .attr("r", defaultR);

  d3.select(`circle.data-index-${el.dataset.index}`)
    .classed("subtle", false)
    .raise()
    .transition()
    .attr("r", bigR);

  if (self) {
    d3.select("." + self.__data__.dist).classed("map-highlight", true);

    d3.select("#date-prominent > span").text(self.__data__.date_str);
    d3.select("#dist-prominent > span").text(self.__data__.dist_name);

    mapZoom(d3.select("." + self.__data__.dist).node());
  }
}

function unhighlight(el) {
  d3.select("#timeline").style("z-index", -1);
  d3.selectAll("circle")
    .classed("subtle", true)
    .transition()
    .attr("r", defaultR);

  d3.select(`circle.data-index-${el.dataset.index}`)
    .classed("subtle", true)
    .transition()
    .attr("r", defaultR);

  d3.selectAll(".map-district").classed("map-highlight", false);

  // mapZoom(null);
}

function changeBackground(className) {
  if (className === "blank-space") return;

  var col = colors[className];

  var hex = col.replace("#", "");
  var r = parseInt(hex.substring(0, 2), 16);
  var g = parseInt(hex.substring(2, 4), 16);
  var b = parseInt(hex.substring(4, 6), 16);

  var result = "rgba(" + r + "," + g + "," + b + "," + 0.5 + ")";

  if (col === "#3a3a3a") {
    result = "#3a3a3a";
  }

  d3.select("body")
    .transition()
    .duration(100)
    .style("background-color", result);

  d3.selectAll(".full-screen")
    .transition()
    .duration(100)
    .style("background-color", result);
}

export default {
  name: "Scroller",
  data: function() {
    return {
      items: [
        {
          message:
            "In 2020, Kashmir witnessed <span class='bold'>98</span> terrorist <span class='bold'>attacks on security forces (SF)</span>, in which <span class='bold red'>29</span> SF personnel and <span class='bold red'>5</span> innocent civilians were killed. The attacks also left <span class='bold gray'>49</span> SF personnel and <span class='bold orange'>38</span> civlians injured.",
          className: "center",
          outerClassName: "center"
        },
        {
          message:
            "In addition to this, there were <span class='bold'>35</span> terrorist <span class='bold'>attacks on civilians</span>, that left <span class='bold red'>27</span> Kashmiris dead and <span class='bold orange'>9</span> injured.",
          className: "center",
          outerClassName: "center"
        },
        {
          message:
            "This is to give you a full picture of what happened in <span class='bold red'>2020</span>.",
          className: "hide-viz",
          outerClassName: "hide-viz",
          index: "hide-viz"
        },
        {
          message: "This is the Kashmir division of J&K",
          className: "highlight-kashmir",
          outerClassName: "highlight-kashmir",
          index: "highlight-kashmir"
        },
        {
          message:
            "The terrorists targetted the very roots of democracy by forever silencing the voices of <span class='bold red'>13 prominent politicians</span> among the 32 civilians killed.",
          className: "democratic",
          index: "democratic",
          outerClassName: "democratic"
        },
        {
          message:
            "Each of these <span class='bold'>133</span> dots represents the tragedy of a terrorist attack in Kashmir in 2020.",
          className: "all-dots",
          outerClassName: "all-dots",
          index: "all-dots"
        },
        {
          message:
            "<span class='bold orange'>Orange</span> dots are attacks where civilians were <span class='bold orange'>injured</span>.",
          className: "injured-dots",
          outerClassName: "injured-dots",
          index: "injured-dots"
        },
        {
          message:
            "<span class='bold red'>Red</span> dots are attacks where civilians were <span class='bold red'>killed</span>.",
          className: "killed-dots",
          outerClassName: "killed-dots",
          index: "killed-dots"
        },
        {
          message:
            "<span class='bold gray'>Gray</span> dots are attacks where fortunately no civilians were <span class='bold gray'>injured or killed.</span> However, they carry the trauma with them.",
          className: "greyed-dots",
          outerClassName: "greyed-dots",
          index: "greyed-dots"
        },
        {
          message:
            "Here are sorrowful stories of some prominent Kashmiris who lost their lives in terrorist attacks in 2020.",
          className: "story-starts center",
          outerClassName: "story-starts",
          index: "story-starts"
        },
        {
          message:
            "<div>On the night of <span class='bold'>1st April 2020</span>, the residents of Nandimarg village in Kulgam District woke up to the sound of multiple gunshots.</div>",
          className: "siraj",
          outerClassName: "siraj",
          index: 22
        },
        {
          message:
            "<div>The victims of the terrorist firing were <span class='bold red'>Sirajuddin Khatana</span>, a BJP party worker, and <span class='bold red'>Gulam Hasan Waghey</span>. Both were residents of Nandimarg.</div>",
          className: "siraj",
          outerClassName: "siraj",
          index: 22
        },
        {
          message: `<div><p class="head">Siraj's family was not new to the loss of loved ones by terrorists. Siraj had lost his father-in-law in July 2009 and his sister and niece later in September of the same year in a similar attack.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/siraj-dead.png"/><p class="caption">Hasan</p></div><div class="image-individual"><img src="images/hasan-dead.png"/><p class="caption">Siraj</p></div></div></div>`,
          index: 22,
          className: "siraj",
          outerClassName: "siraj",
          fullScreen: true
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message:
            "<div>On <span class='bold'>8 June</span>, 40-year-old Congress sarpanch <span class='bold red'>Ajay Pandita</span> was shot dead by terrorists in his orchard in front of his house in Anantnag.</div>",
          className: "pandita-start",
          outerClassName: "pandita",
          index: 49
        },
        {
          message: `<div><p class="head">He was from the minority Kashmiri Pandit community and one of the few who stayed in the valley despite threats.</p><div class="full-screen-image-container"><img src="images/pandita.png"/><p class="caption">Ajay Pandita (right).</p></div></div>`,
          index: 49,
          className: "pandita-video",
          outerClassName: "pandita",
          fullScreen: true
        },
        {
          message: `<div><p class="head"><span class="quote left-quote font-pandita-daughter">&ldquo;</span>I felt proud when my father&#39;s mortal remains were carried in Tricolour. He had seen a dream for India, and now India has to do something for him. My father is "Amar" and "Shaheed".<span class="quote right-quote">&rdquo;</span></p><div class="full-screen-image-container"><img class="pandita-daughter" src="images/pandita-family.png"/><p class="caption">Niyanta Pandita, the brave daughter of late Kashmiri Pandit sarpanch Ajay Pandita (Bharti).</p></div><div><p class="head"><span class="quote left-quote">&ldquo;</span>मैं किसी से डरती नहीं जिन्होंने मेरे पिता के साथ ये किया है उसकी बोटी बोटी अलग कर दूँगी, मैं मेरे पिता का शेर बच्चा हूँ |<span class="quote right-quote">&rdquo;</span></p></div></div>`,
          index: 49,
          className: "pandita-daughter",
          outerClassName: "pandita",
          fullScreen: true
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message:
            "<div>BJP Bandipora president <span class='bold red'>Waseem Bari</span> was a talented young politician who belonged to a family with many years of political background. <span class='bold'>Bari, his brother and father</span> were shot dead on 8 July by bike-borne terrorists.</div>",
          index: 61,
          className: "waseem-bari-start",
          outerClassName: "waseem-bari"
        },
        // {
        //   message: `<div><p class="head"><span class="quote left-quote">&ldquo;</span>In a cowardice act, terrorists from Pakistan took advantage of the dark of the night to attack Bari, his brother and father. He was a patriot, a true nationalist<span class="quote right-quote">&rdquo;</span><span class="quote-by">- Ravinder Raina, BJP state president</span></p><div class="full-screen-image-container"><div class="image-individual"><img src="images/waseem-brother.png"/><p class="caption">Umer Bashir</p></div><div class="image-individual"><img class="waseem-img" src="images/waseem.png"/><p class="caption">Waseem Bari</p></div><div class="image-individual"><img src="images/waseem-father.png"/><p class="caption">Bashir Ahmad</p></div></div></div>`,
        //   index: 61,
        //   fullScreen: true,
        //   className: "waseem-bari",
        //   outerClassName: "waseem-bari"
        // },
        // {
        //   message: `<div><p class="head">Waseem, his brother Umer and father Bashir left behind a mother and a sister.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/waseem-brother.png"/><p class="caption">Umer Bashir</p></div><div class="image-individual"><img class="waseem-img" src="images/waseem.png"/><p class="caption">Waseem Bari</p></div><div class="image-individual"><img src="images/waseem-father.png"/><p class="caption">Bashir Ahmad</p></div></div></div>`,
        //   index: 61,
        //   fullScreen: true,
        //   className: "waseem-bari",
        //   outerClassName: "waseem-bari"
        // },
        {
          message: `<div><p class="head">Waseem, his brother Umer and father Bashir were shot in the head 100 metres away from Bandipora police station.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/waseem-brother.png"/><p class="caption">Umer Bashir</p></div><div class="image-individual"><img class="waseem-img" src="images/waseem.png"/><p class="caption">Waseem Bari</p></div><div class="image-individual"><img src="images/waseem-father.png"/><p class="caption">Bashir Ahmad</p></div></div><div><p class="head">Waseem is survived by his wife, three month old son and a disabled sister.</p></div></div>`,
          index: 61,
          fullScreen: true,
          className: "waseem-bari",
          outerClassName: "waseem-bari"
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message: `On <span class='bold'>6 August</span>, terrorists shot dead BJP sarpanch <span class='bold red'>Sajad Ahmad Khanday</span> in Wissu, Kulgam.`,
          index: 72,
          className: "khanday",
          outerClassName: "khanday"
        },
        {
          message: `<div><p class="head">On the fateful day, he left Wissu migrant camp in the morning to visit his home when the terrorists fired upon him. Khanday was just 20 metres away from his house when he was attacked and killed.</p><div class="full-screen-image-container"><img src="images/khanday.png"/><p class="caption">Sajad Ahmed Khanday</p></div></div>`,
          index: 72,
          fullScreen: true,
          className: "khanday",
          outerClassName: "khanday"
        },
        {
          message: `<div><p class="head">He is survived by his wife and two sons</p><div class="full-screen-image-container"><img src="images/khanday-family.jpg"/><p class="caption">From (left): Sahil, Shahina Begum, Salman</p></div></div>`,
          index: 72,
          fullScreen: true,
          className: "khanday-funeral khanday",
          outerClassName: "khanday"
        },
        {
          message: `<div><p class="head">Large number of people, including senior political leaders, joined the last journey of Khanday.</p><div class="full-screen-image-container"><img src="images/khanday-funeral.png"/><p class="caption">People carrying Khanday's coffin at his village.</p></div></div>`,
          index: 72,
          fullScreen: true,
          className: "khanday-funeral khanday",
          outerClassName: "khanday"
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message: `On <span class='bold'>9 August</span>, terrorists shot dead 35 year old BJP district president of Budgam, <span class='bold red'>Abdul Hamid Najar</span> while he was on a morning walk.`,
          index: 73,
          className: "najar",
          outerClassName: "najar"
        },
        {
          message: `<div><p class="head">Najar was shot in his abdomen and leg, and was shifted to SMHS hospital where he later succumbed to his injuries.</p><div class="full-screen-image-container"><img src="images/najar.png"/><p class="caption"></p></div></div>`,
          index: 73,
          fullScreen: true,
          className: "najar",
          outerClassName: "najar"
        },
        {
          message: `<div><p class="head">He is survived by his wife, mother and daughter.</p><div class="full-screen-image-container"><img src="images/najar-family.png"/><p class="caption"></p></div></div>`,
          index: 73,
          fullScreen: true,
          className: "najar",
          outerClassName: "najar"
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message:
            "<div>On <span class='bold'>24th September</span>, Terrorists fired at 35 year old <span class='bold red'>Babar Qadri</span>, a widely respected advocate living in Srinagar.</div>",
          index: 90,
          className: "babar-qadri",
          outerClassName: "babar-qadri"
        },
        {
          message: `<div><p class="head">Babar Qadri worked for the interest of Kashmir and fought numerous cases for Kashmiri citizens.</p><div class="full-screen-image-container"><img src="images/babar.png"/></div></div>`,
          index: 90,
          fullScreen: true,
          className: "babar-qadri",
          outerClassName: "babar-qadri"
        },
        {
          message: `<div><p class="head">On 24th September, he was live on Facebook and expressed his concern over the future of Kashmir and also shared that he had been threatened.</p><div class="full-screen-image-container"><img src="images/babar-live.png"/></div><p class="head">6 hours later, two unidentified terrorists came to his house and shot him at close range.</p></div>`,
          index: 90,
          fullScreen: true,
          className: "babar-qadri",
          outerClassName: "babar-qadri"
        },
        {
          message: `<div><p class="head">That day whole Kashmir cried as their brave son was killed by terrorists controlled by pro-Pakistan separatists.</p><div class="full-screen-image-container"><img src="images/babar-funeral.png"/></div></div>`,
          index: 90,
          fullScreen: true,
          className: "babar-qadri",
          outerClassName: "babar-qadri"
        },
        {
          message: `<div><p class="head">He is survived by his wife and two daughters.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/babar-family.png"/><p class="caption"></p></div><div class="image-individual"><img class="" src="images/babar-family-2.png"/><p class="caption"></p></div></div></div>`,
          index: 90,
          fullScreen: true,
          className: "babar-qadri",
          outerClassName: "babar-qadri"
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message:
            "<div>On <span class='bold'>6th October</span> in Nunar village in Ganderbal district, Terrorists fired at BJP activist <span class='bold'>Ghulam Qadir</span>. His PSO <span class='bold red'>Altaf Hussain</span> received gunshot wound on his head and was later declared dead.</div>",
          index: 99,
          className: "altaf",
          outerClassName: "altaf"
        },
        {
          message: `<div><p class="head">Altaf is survived by his wife and daughter.</p><div class="full-screen-image-container"><img src="images/altaf.png"/></div></div>`,
          index: 99,
          fullScreen: true,
          className: "altaf",
          outerClassName: "altaf"
        },
        {
          message: `<div><p class="head">Hundreds of people joined funeral prayers for Altaf in Srinagar.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/altaf-funeral.png"/><p class="caption"></p></div><div class="image-individual"><img class="" src="images/altaf-funeral-2.png"/><p class="caption"></p></div><div class="image-individual"><img src="images/altaf-funeral-3.png"/><p class="caption"></p></div></div></div>`,
          index: 99,
          fullScreen: true,
          className: "altaf",
          outerClassName: "altaf"
        },
        {
          message: "",
          className: "blank-space",
          blankSpace: "blank-space",
          outerClassName: "blank-space",
          index: "blank-space"
        },
        {
          message:
            "<div>On 29th October terrorists fired on BJP Yuva Morcha members near YK Pura in Kulgam. The three killed were <span class='bold red'>Fida Hussain Itoo</span>, <span class='bold red'>Umer Rashid Beigh</span> and <span class='bold red'>Umer Hanan</span>.</div>",
          index: 106,
          className: "ykpora",
          outerClassName: "ykpora"
        },
        {
          message: `<div><p class="head">Fida Hussain Itoo is survived by his wife and son.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/ykpora-family-1.png"/><p class="caption">Itoo's family</p></div></div></div>`,
          index: 106,
          fullScreen: true,
          className: "ykpora",
          outerClassName: "ykpora"
        },
        {
          message: `<div><p class="head">Umer Rashid Beigh is survivied by his mother and daughter.</p><div class="full-screen-image-container"><div class="image-individual"><img class="" src="images/ykpora-family-2.png"/><p class="caption">Beigh's family</p></div></div></div>`,
          index: 106,
          fullScreen: true,
          className: "ykpora",
          outerClassName: "ykpora"
        },
        {
          message: `<div><p class="head">Umer Hanan is survived by his wife and daughter.</p><div class="full-screen-image-container"><div class="image-individual"><img src="images/ykpora-family-3.png"/><p class="caption">Hanan's family</p></div></div></div>`,
          index: 106,
          fullScreen: true,
          className: "ykpora",
          outerClassName: "ykpora"
        },
        {
          message:
            "<div>These were just some of the individuals who have lost their lives in terrorist attacks in Kashmir in 2020. You can explore rest of the data by hovering over the dots.</div>",
          className: "last",
          outerClassName: "last",
          index: -1
        }
      ]
    };
  },
  mounted: function() {
    d3.selectAll(".last-show").style("display", "none");
    d3.selectAll(".scroll-section").style(
      "height",
      window.innerHeight * 0.4 + "px"
    );

    d3.selectAll(".full-screen").style("height", window.innerHeight + "px");
    // d3.select("#scroll-container").style("top", "-600px");

    Array.from(
      document.getElementsByClassName("scrolly-text-container")
    ).forEach((el, i, arr) => {
      // if (el.dataset.blankspace === "blank-space") return;
      let elPrev = null;
      let selfPrev = null;

      if (i !== 0) {
        elPrev = arr[i - 1];

        if (elPrev.dataset.index && !isLetter(elPrev.dataset.index[0])) {
          // if (el.dataset.blankspace === "blank-space") console.log(elPrev);
          selfPrev = d3
            .select(`circle.data-index-${elPrev.dataset.index}`)
            .node();
        }
      }
      if (
        el.dataset.index &&
        el.dataset.index !== "-1" &&
        el.dataset.index !== "-2" &&
        !isLetter(el.dataset.index[0])
      ) {
        // console.log(el);
        // eslint-disable-next-line
        new Waypoint({
          element: el,
          handler: function(dir) {
            var self = d3
              .select(`circle.data-index-${el.dataset.index}`)
              .node();

            if (dir === "down") {
              // console.log("here" + 1);
              d3.select("#map-timeline")
                .transition()
                .style("opacity", 1);

              d3.select("#timeline")
                .transition()
                .style("opacity", 1);

              highlight(el, self);

              changeBackground(el.dataset.color);
              if (el.dataset.classname === "siraj") {
                d3.select("#date-dist-prominent")
                  .transition()
                  .style("opacity", 1);
              }
              if (el.dataset.fullscreen === "true") {
                // console.log("here" + 2);
                d3.select("#map-timeline")
                  .transition()
                  .style("opacity", 0);
                d3.select("#timeline")
                  .transition()
                  .style("opacity", 0);
              } else {
                if (elPrev) {
                  if (elPrev.dataset.fullscreen === "true") {
                    // console.log("up, true");

                    // console.log("here" + 3);
                    d3.select("#map-timeline")
                      .transition()
                      .style("opacity", 1);

                    d3.select("#timeline")
                      .transition()
                      .style("opacity", 1);
                  }
                }
              }
            } else {
              unhighlight(el);
              if (elPrev) {
                console.log("here" + 4);
                changeBackground(elPrev.dataset.color);
                if (
                  elPrev.dataset.fullscreen === "true" ||
                  elPrev.dataset.blankspace === "blank-space"
                ) {
                  d3.select("#map-timeline")
                    .transition()
                    .style("opacity", 0);

                  d3.select("#timeline")
                    .transition()
                    .style("opacity", 0);
                }
                if (elPrev.dataset.index === "story-starts") {
                  d3.select("#date-dist-prominent")
                    .transition()
                    .style("opacity", 0);
                }
                // console.log(elPrev, selfPrev);
                highlight(elPrev, selfPrev);
                if (
                  elPrev.dataset.fullscreen !== "true" &&
                  el.dataset.fullscreen === "true"
                ) {
                  // console.log("here" + 5);
                  d3.select("#map-timeline")
                    .transition()
                    .style("opacity", 1);
                  d3.select("#timeline")
                    .transition()
                    .style("opacity", 1);
                }
              }
            }
          },
          offset: "60%"
        });
      } else if (el.dataset.index === "-2") {
        // eslint-disable-next-line
        new Waypoint({
          element: el,
          handler: function(dir) {
            // console.log(colors[el.dataset.color]);
            if (dir === "down") {
              // console.log("here" + 5);
              changeBackground(el.dataset.color);
              // console.log("here" + 6);
              d3.select("#map-timeline")
                .transition()
                .style("opacity", 1);

              d3.select("#timeline")
                .transition()
                .style("opacity", 1);
            } else {
              // console.log("here" + 6);
              changeBackground(elPrev.dataset.color);
              // console.log("here" + 7);
              d3.select("#map-timeline")
                .transition()
                .style("opacity", 0);

              d3.select("#timeline")
                .transition()
                .style("opacity", 0);
            }
          },
          offset: "60%"
        });
      } else if (el.dataset.index === "-1") {
        // eslint-disable-next-line
        new Waypoint({
          element: el,
          handler: function(dir) {
            // console.log(colors[el.dataset.color]);
            if (dir === "down") {
              window.disableMapClick = false;
              changeBackground(el.dataset.color);
              // d3.select("#tooltip")
              //   .transition()
              //   .style("opacity", 1)
              //   .style("display", "block");

              d3.select("#date-dist-prominent")
                .transition()
                .style("opacity", 0);

              d3.selectAll("circle")
                .classed("subtle", false)
                .attr("r", defaultR);

                d3.selectAll(".last-show").style("display", "");

              // d3.selectAll(".last")
              //   .transition()
              //   .style("opacity", 0);

              d3.select("#timeline").style("z-index", 100);

              d3.selectAll(".map-highlight").classed("map-highlight", false);

              // console.log("here" + 8);
              d3.select("#map-timeline")
                .transition()
                .style("opacity", 1);

              d3.select("#timeline")
                .transition()
                .style("opacity", 1);
              // d3.select(".scrolly-text-container.last").classed("fixed", true);
            } else {
              window.disableMapClick = true;
              if (elPrev) {
                changeBackground(elPrev.dataset.color);
              }
              d3.select("#date-dist-prominent")
                .transition()
                .style("opacity", 1);

              d3.select("#tooltip")
                .transition()
                .style("opacity", 0)
                .style("display", "none");

              d3.selectAll("circle")
                .classed("subtle", true)
                .attr("r", defaultR);

                d3.selectAll(".last-show").style("display", "none");

              // d3.selectAll(".last")
              //   .transition()
              //   .style("opacity", 1);

              d3.select("#timeline").style("z-index", -1);

              // console.log("here" + 9);
              d3.select("#map-timeline")
                .transition()
                .style("opacity", 0);

              d3.select("#timeline")
                .transition()
                .style("opacity", 0);
              // d3.select(".scrolly-text-container.last").classed("fixed", false);
            }
          },
          offset: "100px"
        });
      } else {
        if (el.dataset.index === "all-dots") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                d3.select("#timeline")
                  .transition()
                  .style("opacity", 1);
                d3.selectAll("circle").classed("subtle", false);
              } else {
                d3.select("#timeline")
                  .transition()
                  .style("opacity", 0);
                d3.selectAll("circle").classed("subtle", true);
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "hide-viz") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                // console.log("here" + 10);
                d3.select("#map-timeline")
                  .transition()
                  .style("opacity", 1);
                d3.select(".back-img")
                  .transition()
                  .style("opacity", 0);
              } else {
                // console.log("here" + 11);
                d3.select("#map-timeline")
                  .transition()
                  .style("opacity", 0);

                d3.select(".back-img")
                  .transition()
                  .style("opacity", 1);
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "story-starts") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                // console.log("here" + 12);
                d3.select("#map-timeline")
                  .transition()
                  .style("opacity", 1);
              } else {
                //
                // console.log("here" + 13);
                d3.select("#map-timeline");
                console.log("here" + 7);
                changeBackground(elPrev.dataset.color);
                //   .transition()
                //   .style("opacity", 1);

                d3.selectAll(".attack-simple")
                  .raise()
                  .transition()
                  .attr("r", bigR);
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "blank-space") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              if (dir === "down") {
                // console.log("here" + 14 + "down");
                // d3.select("#map-timeline")
                //   .transition()
                //   .style("opacity", 1);
              } else {
                if (elPrev) {
                  // console.log("here" + 14 + "up");
                  console.log("here" + 8);
                  changeBackground(elPrev.dataset.color);
                  // console.log(el, elPrev, selfPrev);
                  if (!selfPrev) {
                    if (
                      elPrev.dataset.index &&
                      !isLetter(elPrev.dataset.index[0])
                    ) {
                      selfPrev = d3
                        .select(`circle.data-index-${elPrev.dataset.index}`)
                        .node();
                    }
                  }
                  highlight(elPrev, selfPrev);
                }
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "highlight-kashmir") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                jammuDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#4d4d4d");
                });

                kashmirDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#fc6363");
                });
                d3.select(".jnk-label").classed("dark-greyed", false);
              } else {
                jammuDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#ccc");
                });
                kashmirDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#ccc");
                });
                d3.select(".jnk-label").classed("dark-greyed", true);
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "injured-dots") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                d3.selectAll("circle").classed("subtle", true);

                d3.selectAll(".civ-injured")
                  .classed("subtle", false)
                  .raise()
                  .transition()
                  .attr("r", bigR);
              } else {
                d3.selectAll("circle").classed("subtle", false);
                d3.selectAll(".civ-injured")

                  .raise()
                  .transition()
                  .attr("r", defaultR);
              }
            },
            offset: "60%"
          });
        } else if (el.dataset.index === "democratic") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                kashmirDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#ccc");
                });
              } else {
                kashmirDistricts.forEach((selector, i) => {
                  d3.select(selector)
                    .transition()
                    .delay(20 * i)
                    .style("fill", "#fc6363");
                });
              }
            },
            offset: "60%"
          });
        }
        if (el.dataset.index === "killed-dots") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                d3.selectAll(".civ-injured")
                  .classed("subtle", true)
                  .transition()
                  .attr("r", defaultR);

                d3.selectAll(".civ-killed")
                  .classed("subtle", false)
                  .raise()
                  .transition()
                  .attr("r", bigR);
              } else {
                d3.selectAll(".civ-killed")
                  .classed("subtle", true)
                  .transition()
                  .attr("r", defaultR);

                d3.selectAll(".civ-injured")
                  .classed("subtle", false)
                  .raise()
                  .transition()
                  .attr("r", bigR);
              }
            },
            offset: "60%"
          });
        }
        if (el.dataset.index === "greyed-dots") {
          // eslint-disable-next-line
          new Waypoint({
            element: el,
            handler: function(dir) {
              // console.log(colors[el.dataset.color]);
              if (dir === "down") {
                d3.selectAll(".civ-killed")
                  .transition()
                  .attr("r", defaultR);

                d3.selectAll(".attack-simple")
                  .raise()
                  .transition()
                  .attr("r", bigR);
              } else {
                d3.selectAll(".civ-killed")
                  .raise()
                  .transition()
                  .attr("r", bigR);

                d3.selectAll(".attack-simple")
                  .transition()
                  .attr("r", defaultR);
              }
            },
            offset: "60%"
          });
        }
      }
    });
  }
};
</script>

<style lang="scss">
$red: rgb(255, 162, 162);
$orange: rgb(252, 208, 133);
$gray: #ccc;

#scroll-container {
  position: relative;
  top: -180vh;
  .image-container {
    display: inline;
  }
  width: 100vw;
  .scroll-section {
    // height: 70vh;

    &.full-screen {
      // height: 100vh;
    }

    width: 100%;
    .scrolly-text-container {
      &.blank-space {
        opacity: 0;
        .scrolly-text-container {
        }
      }
      font-size: 1.3rem;
      .scrolly-text {
        > div {
          position: relative;
        }
      }

      &:not(.full-screen) {
        .scrolly-text {
          border: 2.5px solid #b34b4b;
          padding: 10px;
          border-radius: 5px;
          line-height: 2.2rem;
        }
      }

      &.pandita,
      &.khanday,
      &.nisa-ah-bhat,
      &.waseem-bari {
        text-align: center;
        width: 600px;
        img {
          height: 300px;
          margin: 10px;
        }
      }

      &.babar-qadri-kids {
        img {
          height: 600px !important;
          width: auto !important;
        }
      }
      &.pandita-video {
        width: 600px;
        pointer-events: all;
      }
      &.center {
        margin-right: auto;
      }

      width: 500px;
      // height: 150px;
      background: #ffffffdd;
      margin-top: 200px;
      margin-left: auto;
      margin-right: 150px;
      border-radius: 5px;
      padding: 10px;
      border: 1px solid #3a3a3a;

      &.story-starts {
        font-weight: bold;
        width: 90%;
        height: 100px;
        position: relative;
        border: 2.5px solid #b34b4b;
        border-radius: 5px;
        z-index: 1000;

        .scrolly-text {
          text-align: center;
          position: absolute;
          border: none;
          padding: 0px;
          top: 50%;
          left: 50%;
          -ms-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
        }
      }

      &.full-screen {
        background-color: #2c2b2b;
        width: 100vw;
        height: 100vh;
        text-align: center;
        position: relative;
        z-index: 1000;

        .full-screen-image-container {
          display: inline-block;
          .image-individual {
            display: inline-block;
            margin: 10px;
            margin-bottom: 0px;
          }
        }

        &.siraj,
        &.waseem-bari,
        &.ykpora,
        &.altaf,
        &.babar-qadri {
          .image-individual {
            // width: 200px;
            vertical-align: middle;
          }
          img {
            height: 300px;
            width: auto;
            margin-bottom: 0px;
          }
        }

        &.najar {
          .image-individual {
            // width: 200px;
            vertical-align: middle;
          }
          img {
            height: 470px;
            width: auto;
            margin-bottom: 0px;
          }
        }

        &.waseem-bari {
          img {
            height: 220px;
          }
          .waseem-img {
            height: 300px;
          }
        }

        p.head {
          color: white;
          font-family: "Playfair Display", serif;
          font-size: 2rem;
          max-width: 900px;
          margin: 50px auto;
          position: relative;

          .quote {
            font-size: 10rem;
            position: absolute;
            color: #b34b4b;
            z-index: 1000;
          }
          .right-quote {
            bottom: -120px;
            // right: -25px;
          }
          .left-quote {
            left: -50px;
            top: -80px;
          }
          .quote-by {
            font-size: 0.9rem;
            font-style: italic;
            color: #aaa;
            position: absolute;
            bottom: -30%;
            left: 50%;
            -ms-transform: translate(-50%, 0%);
            transform: translate(-50%, 0%);
          }
        }

        p.caption {
          color: white !important;
          font-size: 0.9rem;
          // width: 500px;
          margin: auto;
          text-align: left;
          color: gray;
          padding-left: 10px;
        }

        img {
          width: 500px;
          height: auto;
          // margin-top: 50px;
          border: 4px solid #993f3f;
          padding: 10px;

          background-color: #ffffff97;
          border-radius: 5px;
        }

        &.khanday {
          img {
            height: 350px;
            width: auto;
          }
          p.caption {
            width: 350px;
            text-align: center;
          }
        }
        &.pandita-daughter {
          .full-screen-image-container {
            margin: 5px;
          }
          img {
            height: 320px;
            width: auto;
          }
          p.head {
            font-size: 1.5rem !important;
          }
        }
      }
    }
  }
}

@media (max-width: 500px) {
  #scroll-container {
    max-width: 300px;
    margin: auto;
    .scroll-section {
      width: 100%;
      .scrolly-text-container {
        max-width: 300px;
        font-size: 1.3rem;
        .scrolly-text {
          > div {
            position: relative;
          }
        }
        &:not(.full-screen) {
          .scrolly-text {
            border: 2.5px solid #b34b4b;
            padding: 10px;
            border-radius: 5px;
          }
        }

        &.pandita,
        &.khanday,
        &.nisa-ah-bhat,
        &.waseem-bari {
          text-align: center;
          width: 600px;
          img {
            height: 300px;
            margin: 10px;
          }
        }

        &.babar-qadri-kids {
          img {
            height: 600px !important;
            width: auto !important;
          }
        }
        &.pandita-video {
          width: 600px;
        }
        &.center {
          margin-right: auto;
        }

        width: 500px;

        &.story-starts {
          height: auto;
          border: none;

          .scrolly-text {
            text-align: center;
            position: static;
            transform: none;
          }
        }

        &.full-screen {
          background-color: #2c2b2b;
          width: 100vw;
          height: 100vh;
          text-align: center;
          position: relative;
          z-index: 1000;

          .full-screen-image-container {
            display: inline-block;
            .image-individual {
              display: inline-block;
              margin: 10px;
              margin-bottom: 0px;
            }
          }

          &.siraj,
          &.waseem-bari,
          &.altaf {
            .image-individual {
              // width: 200px;
              vertical-align: middle;
            }
            img {
              height: 180px;
              width: auto;
              margin-bottom: 0px;
            }
          }

          &.ykpora {
            .image-individual {
              // width: 200px;
              vertical-align: middle;
            }
            img {
              height: auto;
              width: 100%;
            }
          }

          &.babar-qadri {
            .full-screen-image-container {
              img {
                width: 100%;
                height: auto;
              }
              .image-individual {
                img {
                  width: 100% !important;
                  height: auto !important;
                }
              }
            }
          }

          &.najar {
            .image-individual {
              // width: 200px;
              vertical-align: middle;
            }
            .full-screen-image-container {
            }
            img {
              width: 100% !important;
              height: auto;
              margin-bottom: 0px;
            }
          }

          &.pandita-daughter {
            .full-screen-image-container {
              margin: 20px;
            }
            img {
              height: 200px;
              width: auto;
            }
            p.head {
            font-size: 1.1rem !important;
          }
          }
          &.waseem-bari {
            .full-screen-image-container {
              margin-top: 50px;
              .image-individual {
                margin: 0;
              }
              img {
                height: 150px !important;
              }
            }
          }

          &.khanday {
            .full-screen-image-container {
              width: 100%;
              margin-top: 50px;
              img {
                width: 100%;
                height: auto;
                margin: 0px;
              }
              p.caption {
                width: 100%;
                text-align: center;
                padding-left: 0px;
              }
            }
          }

          p.head {
            color: white;
            font-family: "Playfair Display", serif;
            font-size: 1.1rem;
            max-width: 900px;
            margin: 10px auto;
            position: relative;
            .quote {
              font-size: 10rem;
              position: absolute;
              color: #b34b4b;
              z-index: -1;
            }
            .right-quote {
              bottom: -120px;
              // right: -25px;
            }
            .left-quote {
              left: -50px;
              top: -80px;
            }
            .quote-by {
              font-size: 0.9rem;
              font-style: italic;
              color: #aaa;
              position: absolute;
              bottom: -40%;
              left: 50%;
              -ms-transform: translate(-50%, 0%);
              transform: translate(-50%, 0%);
            }
          }

          p.caption {
            font-size: 0.9rem;
            // width: 500px;
            margin: auto;
            text-align: left;
            color: gray;
            padding-left: 10px;
          }

          img {
            width: 100%;
            height: auto;
            // margin-top: 50px;
            border: 4px solid #993f3f;
            padding: 0px;

            background-color: #ffffff97;
            border-radius: 5px;
          }
        }
      }
    }
  }
}

.last {
  &.fixed {
    position: fixed;
    top: 60%;
    right: 150px;
  }
}

span {
  &.bold {
    font-weight: bold !important;
  }

  &.red {
    white-space: nowrap;
    padding: 1px 6px;
    // margin: 1px 2px;
    border-radius: 7px;
    background-color: rgba($red, 0.7) !important;
    border: 2px solid $red;
  }

  &.orange {
    white-space: nowrap;
    padding: 1px 6px;
    // margin: 1px 2px;
    border-radius: 7px;
    background-color: rgba($orange, 0.7) !important;
    border: 2px solid $orange;
  }

  &.gray {
    white-space: nowrap;
    padding: 1px 6px;
    // margin: 1px 2px;
    border-radius: 7px;
    background-color: rgba($gray, 0.7) !important;
    border: 2px solid $gray;
  }
}

</style>
