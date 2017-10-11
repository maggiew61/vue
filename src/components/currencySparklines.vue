<template>
  <div class="currencySparklines" :style="{ width: width + 'px' , height: height + 'px'  }" >
    <transition name="fade">
      <div :ref="id" class="chartPlace" :data-id="id" :style="{visibility: loading ? 'hidden' : 'visible'  }">
      </div>
    </transition>
  </div>
</template>

<script>

import * as d3 from 'd3';

export default {
  name: 'currencySparklines',
  data() {
    return {
      loading: false,
      el: null,
    };
  },
  props: {
    data: {
      type: Array,
      default: [],
    },
    width: {
      type: Number,
      default: 90,
    },
    height: {
      type: Number,
      default: 21,
    },
    id: {
      type: String,
      default: Math.random * 1000,
      required: true,
    },
  },
  mounted() {
    this.init();
    this.renderChart();
  },
  watch: {
    data() {
      this.renderChart();
    },
  },
  computed: {
    chartData() {
      const data = [];
      for (let i = 0; i < this.data.length; i += 1) {
        data.push({ name: this.data[i][0], value: this.data[i][1] });
      }
      return data;
    },
  },
  methods: {
    init() {
      this.el = d3.select(this.$refs[this.id]);
    },
    tooltip(data) {
      console.log(data);
    },
    renderChart() {
      const id = this.id;
      const el = this.el;
      el.html('');
      const BBRect = el.node().getBoundingClientRect();
      const width = BBRect.width;
      const height = BBRect.height;
      const chartData = this.chartData;
      const lineColor = d3.scaleOrdinal(d3.schemeCategory10);
      const margin = {
        top: 4,
        bottom: 4,
        right: 4,
        left: 4,
      };
      const regionWidth = width - margin.right - margin.left;
      const regionHeight = height - margin.top - margin.bottom;
      // const total = d3.sum(chartData, (data => d3.sum(data, d => d.value)));
      const maxValue = d3.max(chartData, (d => d.value));
      const minValue = d3.min(chartData, (d => d.value));
      function translation(x, y) {
        return `translate(${x}, ${y})`;
      }

      let yScaleMax = maxValue * 1;
      let yScaleMin = minValue / 1.001;
      if (minValue < 0) {
        yScaleMax = maxValue / 1;
        yScaleMin = minValue * 1.001;
      }
      const xScale = d3.scaleLinear()
        .domain([0, chartData.length - 1])
        .range([0, regionWidth]);
      const yScale = d3.scaleLinear()
        .domain([yScaleMax, yScaleMin])
        .range([0, regionHeight]);

      const line = d3.line()
        .x((d, i) => xScale(i))
        .y(d => yScale(d.value));

      const area = d3.area()
        .x((d, i) => xScale(i))
        .y0(regionHeight)
        .y1(d => yScale(d.value));

      const div = el.append('div')
        .classed('BrokenLine', true);
      const svg = div.append('svg')
        .attr('width', width + margin.right + margin.left)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', translation(margin.left, margin.top));

      function drawPath(selector, data) {
        svg.append('path')
          .classed(selector, true)
          .attr('d', line(data))
          .attr('y', 0)
          .attr('transform', 'translate(0,0)')
          .style('stroke', "transparent'")
          .style('stroke-width', '1px')
          .style('fill', 'transparent')
          .transition()
          .duration(400)
          .delay(0)
          .ease(d3.easeLinear)
          .style('stroke', lineColor(data));

        svg.append('path')
             .data([data])
             .attr('class', 'area')
             .attr('d', area)
             .style('fill', 'transparent')
             .transition()
             .duration(400)
             .delay(0)
             .ease(d3.easeLinear)
             .style('fill', 'rgb(193, 219, 236)');

        svg.selectAll('line')
            .data(data)
            .enter().append('line')
            .attr('class', (d, i) => 'line line-' + i)
            .attr('x1', (d, i) => xScale(i))
            .attr('y1', 0)
            .attr('x2', (d, i) => xScale(i))
            .attr('y2', regionHeight)
            .style('stroke', 'transparent')
            .style('stroke-width', 1)
            .style('cursor', 'pointer')
            .on('mouseover', function (d, i) {
              d3.select(this)
                .style('stroke', '#9e9e9e');
              d3.select(`.chartPlace[data-id="${id}"] .circle-${i}`)
                .style('r', 3)
                .style('opacity', 0.8);
            })
            .on('mouseout', function (d, i) {
              d3.select(this)
                .style('stroke', 'transparent');
              d3.select(`.chartPlace[data-id=${id}] .circle-${i}`)
                .style('r', 1.5)
                .style('opacity', 1);
            });

        svg.selectAll('dot')
          .data(data)
          .enter().append('circle')
          .style('fill', lineColor(data))
          .style('cursor', 'pointer')
          .attr('class', (d, i) => 'circle circle-' + i)
          .attr('cx', (d, i) => xScale(i))
          .attr('cy', d => yScale(d.value))
          .attr('r', 1.5)
          .on('mouseover', function (d, i) {
            d3.select(`.chartPlace[data-id="${id}"] .line-${i}`)
              .style('stroke', '#9e9e9e');
            d3.select(this)
              .style('r', 3)
              .style('opacity', 0.8);
            d3.select('.d3tooltip')
              .html(`${d.name} : ${d.value}`)
              .style('display', 'block')
              .style('left', (this.getBoundingClientRect().left + 10) + 'px')
              .style('top', (this.getBoundingClientRect().top - 20) + 'px');
          })
          .on('mouseout', function (d, i) {
            d3.select(this)
            .style('r', 1.5)
            .style('opacity', 1);
            d3.select(`.chartPlace[data-id=${id}] .line-${i}`)
              .style('stroke', 'transparent');
            d3.select('.d3tooltip')
              .style('display', 'none');
          });
      }
      drawPath('bl-data-01', chartData);
    },
  },
};
</script>

<style scoped lang="scss">
  .chartPlace{
    width: inherit;
    height: inherit;
  }

</style>
