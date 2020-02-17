<template>
  <div class="chord">
    <svg :viewBox='viewBox' :width='width' :height='height'>
      <g>
        <g :key='group.index' v-for='group in chords.groups'>
          <path :fill='color(group.index)' :stroke='color(group.index)' :d='arc(group)'></path>
        </g>
      </g>
      <g fill-opacity='0.67'>
        <path :key='ch.source.index' v-for='ch in chords' :fill='color(ch.source.index)' :stroke='color(ch.source.index)' :d='ribbon(ch)'></path>
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3';

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
        // [11975,  5871, 8916, 2868],
        // [ 1951, 10048, 2060, 6171],
        // [ 8010, 16145, 8090, 8045],
        // [ 1013,   990,  940, 6907]
      ],
    }
  },
  computed: {
    viewBox: function() {
      return [-this.width / 2, -this.height / 2, this.width, this.height];
    },
    chords: function() {
      const chords = this.chord(this.matrix);
      return chords;
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
      console.log("Power set: " + powerSet);
      return powerSet;
    },
  },
  created() {
    this.formMatrix();
  },
  methods: {
    formMatrix() {
      console.log(this.powerSet);
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
      console.log(this.matrix);
    },
    chord: function(data) {
      return d3.chord()
      .padAngle(.04)
      .sortSubgroups(d3.descending)
      .sortChords(d3.descending)(data);
    },
    arc: function(data) {
      const arc = d3.arc()
      .innerRadius(this.innerRadius)
      .outerRadius(this.innerRadius + 20);
      return arc(data);
    },
    color: function(index) {
      console.log(index);
      const c = d3.schemeCategory10[index];
      console.log(c);
      return c;
    },
    ribbon: function(data) {
      const ribbon = d3.ribbon()
      .radius(this.innerRadius);
      return ribbon(data);
    },
    charToNumber(char) {
      console.log(char);
      return char.charCodeAt(0) - 65;
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
