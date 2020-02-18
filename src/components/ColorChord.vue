<template>
  <div class="chord">
    <svg :viewBox='viewBox' :width='width' :height='height'>
      <g>
        <g :key='group.index' v-for='group in groups' :fill-opacity='group.active ? 1 : 0.3' @click="clickGroup(group)">
          <path :fill='groupColor(group)' :stroke='groupColor(group)' :d='arc(group)'></path>
        </g>
      </g>
      <g fill-opacity='0.67'>
        <defs>
          <linearGradient
                  :key='chordKey(ch, "grad")'
                  v-for='ch in chords'
                  :id='chordKey(ch, "grad_id")'
                  gradientUnits='userSpaceOnUse'
                  :x1='gradient_x(ch.source)'
                  :y1='gradient_y(ch.source)'
                  :x2='gradient_x(ch.target)'
                  :y2='gradient_y(ch.target)'
          >
            <stop offset="0%" :stop-color="chordColor(ch.source, ch.active)"></stop>
            <stop offset="100%" :stop-color="chordColor(ch.target, ch.active)"></stop>
          </linearGradient>
        </defs>
        <path :key='chordKey(ch)' v-for='ch in chords' :fill='gradient_fill(ch)' :d='ribbon(ch)'></path>
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3';
import { colors } from "../constants";

export default {
  name: 'ColorChord',
  data() {
    return {
      width: 700,
      height: 700,
      palettes: [
        'DB', 'LN', 'KG', 'HJ',
        'NA', 'OF', 'IE', 'GB',
        'HBD', 'EFH', 'KGN', 'KJM',
        'CEL', 'ONA', 'FGJ', 'LOI',
        'KHOL', 'CHGK', 'CALJ', 'NAGK',
        'NCDH', 'KJMG', 'ILHE', 'CDNM'
        ],
      matrix: [
      ],
      groups: [],
      chords: [],
    }
  },
  computed: {
    viewBox: function() {
      return [-this.width / 2, -this.height / 2, this.width, this.height];
    },
    outerRadius: function() {
      return Math.min(this.width, this.height) * 0.5;
    },
    innerRadius: function() {
      return this.outerRadius - 124;
    },
        powerSet() {
      let powerSet = [];
      this.palettes.forEach(palette => {
        if (palette.length > 2) {
          for (let i = 0; i < palette.length; i++) {
            const firstChar = palette.charAt(i);
            for (let j = i + 1; j < palette.length; j++) {
              const secondChar = palette.charAt(j);
              powerSet.push(`${firstChar}${secondChar}`);
            }
          }
        } else {
          powerSet.push(palette);
        }
      });
      return powerSet;
    },
  },
  created() {
    this.formMatrix();

    const chords = this.chord(this.matrix);
    this.groups = chords.groups.map(group => ({...group, active: false}));
    this.chords = chords.map(chord => ({...chord, active: false}));
  },
  methods: {
    formMatrix() {
      this.powerSet.forEach(pair => {
        const index1 = this.charToNumber(pair.charAt(0));
        const index2 = this.charToNumber(pair.charAt(1));
        let row1 = this.matrix[index1];
        if (!row1) row1 = this.matrix[index1] = Array.from({length: 15}).fill(0);
        let row2 = this.matrix[index2];
        if (!row2) row2 = this.matrix[index2] = Array.from({length: 15}).fill(0);
        row1[index2]++;
        row2[index1]++;
      });
    },
    chord: function(data) {
      return d3.chord()
      .padAngle(.04)
      .sortGroups(d3.descending)
      .sortChords(d3.descending)(data);
    },
    arc: function(data) {
      const arc = d3.arc()
      .innerRadius(this.innerRadius)
      .outerRadius(this.innerRadius + 20);
      return arc(data);
    },
    groupColor: function(group) {
      const index = group.index;
      let color = colors[this.numberToChar(index)];
      return color;
    },
    chordColor: function(data, active=false) {
      active = data.active ? data.active !== undefined : active;
      if (active) {
        const index = data.index;
        return  colors[this.numberToChar(index)];
      } else {
        return '#cccccc';
      }
    },
    gradient_fill: function(chord) {
      return `url(#${this.chordKey(chord, 'grad_id')})`
    },
    ribbon: function(data) {
      const ribbon = d3.ribbon()
      .radius(this.innerRadius);
      return ribbon(data);
    },
    gradient_x: function(node_data) {
        return this.innerRadius * Math.cos((node_data.endAngle-node_data.startAngle)/2 +
        node_data.startAngle-Math.PI/2);
    },
    gradient_y: function(node_data) {
        return this.innerRadius*Math.sin((node_data.endAngle-node_data.startAngle)/2 +
        node_data.startAngle-Math.PI/2);
    },
    charToNumber(char) {
      return char.charCodeAt(0) - 65;
    },
    numberToChar(num) {
      return String.fromCharCode(num + 65);
    },
    chordKey(chord, prefix='ch') {
      return `${prefix}_${chord.source.index}_${chord.target.index}`;
    },
    clickGroup(group) {
      const toggle = !group.active;
      group.active = toggle;
      this.chords.forEach(chord => {
        if (chord.source.index === group.index) {
          chord.active = toggle;
        }
      });
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
