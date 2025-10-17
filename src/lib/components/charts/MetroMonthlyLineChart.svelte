<script lang="ts">
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  /**
   * MetroMonthlyLineChart
   * Enhanced compact horizontal linear line chart for metro monthly data.
   * - Clean design with smooth animations
   * - No numeric labels displayed
   * - Progress prop (0→1) reveals the line progressively from left to right
   * - Responsive and mobile-friendly
   */
  export let data: Array<
    { Yıl?: string | number; Vaka?: number } | Record<string, any>
  > = [];
  export let width: number = 520;
  export let height: number = 280;
  export let title: string = "Metro İstanbul'da aylara göre vakalar";
  export let progress: number = 1; // 0..1

  let container: HTMLDivElement;
  let svg: d3.Selection<SVGSVGElement, unknown, null, undefined> | null = null;
  let lineSel: d3.Selection<SVGPathElement, unknown, null, undefined> | null =
    null;
  let dotsSel: any = null;

  onMount(() => {
    draw();
  });

  $: if (data && container) {
    draw();
  }

  $: if (svg && progress !== undefined) {
    animateLine();
  }

  function parseData() {
    const parsed = data
      .map((d: any, i: number) => ({
        idx: i,
        label: String(d.Yıl ?? d.yil ?? d.month ?? d.label ?? i + 1),
        value: Number(d.Vaka ?? d.vaka ?? d.value ?? 0),
      }))
      .filter((d) => !Number.isNaN(d.value));
    return parsed;
  }

  function draw() {
    if (!container) return;

    const series = parseData();
    if (series.length === 0) return;

    const margin = { top: 32, right: 20, bottom: 40, left: 20 };
    const w = Math.max(320, width);
    const h = Math.max(180, height);
    const cw = w - margin.left - margin.right;
    const ch = h - margin.top - margin.bottom;

    d3.select(container).selectAll('*').remove();

    svg = d3
      .select(container)
      .append('svg')
      .attr('width', w)
      .attr('height', h)
      .attr('viewBox', `0 0 ${w} ${h}`)
      .attr('preserveAspectRatio', 'xMidYMid meet');

    const g = svg
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    // Enhanced scales with better domain handling
    const x = d3
      .scaleLinear()
      .domain([0, Math.max(1, series.length - 1)])
      .range([0, cw]);

    const maxValue = d3.max(series, (d) => d.value) ?? 1;
    const y = d3.scaleLinear().domain([0, maxValue]).nice().range([ch, 0]);

    // Smooth curve for better visual appeal
    const line = d3
      .line<{ idx: number; value: number }>()
      .x((d) => x(d.idx))
      .y((d) => y(d.value))
      .curve(d3.curveCatmullRom.alpha(0.5));

    // Gradient definition for the line
    const defs = svg.append('defs');
    const gradient = defs
      .append('linearGradient')
      .attr('id', 'lineGradient')
      .attr('x1', '0%')
      .attr('y1', '0%')
      .attr('x2', '100%')
      .attr('y2', '0%');

    gradient
      .append('stop')
      .attr('offset', '0%')
      .attr('stop-color', '#d32f2f')
      .attr('stop-opacity', 0.8);

    gradient
      .append('stop')
      .attr('offset', '100%')
      .attr('stop-color', '#b71c1c')
      .attr('stop-opacity', 1);

    // Background grid (subtle)
    g.append('g')
      .attr('class', 'grid')
      .attr('transform', `translate(0,${ch})`)
      .call(
        d3
          .axisBottom(x)
          .tickSize(-ch)
          .tickFormat(() => '')
      )
      .style('stroke', '#f1f5f9')
      .style('stroke-width', 1)
      .style('opacity', 0.6);

    // Main line path
    lineSel = g
      .append('path')
      .datum(series)
      .attr('class', 'trend-line')
      .attr('fill', 'none')
      .attr('stroke', 'url(#lineGradient)')
      .attr('stroke-width', 3)
      .attr('stroke-linecap', 'round')
      .attr('stroke-linejoin', 'round')
      .attr('d', line as any);

    // Enhanced dots with hover effects
    dotsSel = g
      .selectAll('.dot')
      .data(series)
      .enter()
      .append('circle')
      .attr('class', 'dot')
      .attr('cx', (d) => x(d.idx))
      .attr('cy', (d) => y(d.value))
      .attr('r', 4)
      .attr('fill', '#d32f2f')
      .attr('stroke', '#ffffff')
      .attr('stroke-width', 2)
      .attr('opacity', 0)
      .style('cursor', 'pointer');

    // Title with better typography
    svg
      .append('text')
      .attr('class', 'chart-title')
      .attr('x', w / 2)
      .attr('y', 20)
      .attr('text-anchor', 'middle')
      .style('font-family', "'Helvetica Neue', Arial, sans-serif")
      .style('font-size', '16px')
      .style('font-weight', 600)
      .style('fill', '#1f2937')
      .style('letter-spacing', '-0.3px')
      .text(title);

    // Cleaner month labels with better spacing
    const sampleEvery = Math.max(1, Math.ceil(series.length / 8));
    g.selectAll('.xlabel')
      .data(
        series.filter(
          (_, i) => i % sampleEvery === 0 || i === series.length - 1
        )
      )
      .enter()
      .append('text')
      .attr('class', 'xlabel')
      .attr('x', (d) => x(d.idx))
      .attr('y', ch + 24)
      .attr('text-anchor', 'middle')
      .style('font-family', "'Helvetica Neue', Arial, sans-serif")
      .style('font-size', '11px')
      .style('font-weight', 500)
      .style('fill', '#6b7280')
      .style('letter-spacing', '-0.2px')
      .text((d) => d.label);

    animateLine();
  }

  function animateLine() {
    if (!lineSel || !dotsSel) return;

    const total = (lineSel.node() as SVGPathElement).getTotalLength?.() || 0;
    const currentProgress = Math.max(0, Math.min(1, progress));

    // Animate line reveal
    lineSel
      .attr('stroke-dasharray', `${total} ${total}`)
      .attr('stroke-dashoffset', total - total * currentProgress);

    // Animate dots appearance
    dotsSel.each(function (d, i) {
      const dot = d3.select(this);
      const dotProgress = (i + 1) / dotsSel.size();

      if (currentProgress >= 1) {
        dot
          .transition()
          .duration(300)
          .delay(i * 50)
          .attr('opacity', 1)
          .attr('r', 4);
      } else if (currentProgress >= dotProgress * 0.8) {
        dot.transition().duration(200).attr('opacity', 1).attr('r', 4);
      } else {
        dot.attr('opacity', 0).attr('r', 0);
      }
    });
  }
</script>

<div class="metro-monthly-line" bind:this={container}></div>

<style>
  .metro-monthly-line {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background: transparent;
    border-radius: 8px;
    padding: 0.5rem;
  }

  /* Enhanced hover effects for dots */
  :global(.metro-monthly-line .dot:hover) {
    r: 6 !important;
    stroke-width: 3 !important;
    transition: all 0.2s ease;
  }

  /* Smooth transitions for all elements with cubic-bezier easing */
  :global(.metro-monthly-line .trend-line) {
    transition: stroke-dashoffset 0.15s cubic-bezier(0.4, 0, 0.2, 1);
  }

  :global(.metro-monthly-line .dot) {
    transition:
      opacity 0.3s cubic-bezier(0.4, 0, 0.2, 1),
      r 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  /* Responsive adjustments */
  @media (max-width: 768px) {
    .metro-monthly-line {
      padding: 0.25rem;
    }

    :global(.metro-monthly-line .chart-title) {
      font-size: 14px !important;
    }

    :global(.metro-monthly-line .xlabel) {
      font-size: 10px !important;
    }

    :global(.metro-monthly-line .dot) {
      r: 3 !important;
    }
  }

  @media (max-width: 480px) {
    :global(.metro-monthly-line .chart-title) {
      font-size: 13px !important;
    }

    :global(.metro-monthly-line .xlabel) {
      font-size: 9px !important;
    }

    :global(.metro-monthly-line .trend-line) {
      stroke-width: 2.5 !important;
    }
  }
</style>
