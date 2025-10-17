<!--
  GlobalSolutionsSwarm.svelte
  Pure SVG + D3 force simulation for Global Examples (STEP 18.1–18.5)
  Pattern: Ipsos-style swarmytelling with scene-based clustering
-->

<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import * as d3 from 'd3';
  import {
    chartTransition,
    type BubbleNode,
  } from '$lib/stores/chartTransition';
  import type { GlobalSolutionRow } from '$lib/utils/csvParser';

  export let width = 900;
  export let height = 420;
  export let stepId: string = 'step-18.1';
  export let data: GlobalSolutionRow[] = [];
  export let cityFilter: string | null = null; // null => all cities
  export let highlightedCity: string | null = null; // city to highlight

  let svgEl: SVGSVGElement;
  let svg: d3.Selection<SVGSVGElement, unknown, null, undefined>;
  let simulation: d3.Simulation<Node, undefined>;

  // Country color palette - expanded for all 9 cities
  const COUNTRY_COLORS: Record<string, string> = {
    Singapore: '#f39c12', // Singapore Orange
    Tokyo: '#e74c3c', // Japan Red
    London: '#2c3e50', // UK Dark Blue
    Seoul: '#3498db', // Korea Blue
    Paris: '#9b59b6', // France Purple
    Toronto: '#e67e22', // Canada Orange
    Berlin: '#34495e', // Germany Dark Gray
    Amsterdam: '#8e44ad', // Netherlands Purple
  };

  const COUNTRY_NAMES: Record<string, string> = {
    Singapore: 'Singapur',
    Tokyo: 'Tokyo',
    London: 'Londra',
    Seoul: 'Seul',
    Paris: 'Paris',
    Toronto: 'Toronto',
    Berlin: 'Berlin',
    Amsterdam: 'Amsterdam',
  };

  type MeasureType = 'door' | 'pit' | 'led' | 'training' | 'helpline';

  type Node = {
    id: number;
    type: MeasureType;
    city: string;
    x: number;
    y: number;
    vx?: number;
    vy?: number;
    radius: number;
    effectiveness: number; // Store effectiveness percentage
    year: number;
    station: string;
  };

  let nodes: Node[] = [];
  let labelRects: Array<{
    city: string;
    x1: number;
    y1: number;
    x2: number;
    y2: number;
  }> = [];

  // Hover state for showing details
  let hoveredNode: Node | null = null;
  let mouseX = 0;
  let mouseY = 0;

  // Three-tiered size system (üç kademeli)
  // Small, Medium, Large categories with clear size gaps
  const SIZE_CONFIG: Record<MeasureType, { base: number; variation: number }> =
    {
      door: { base: 9, variation: 1.5 }, // LARGE - Platform Doors (most important)
      pit: { base: 2, variation: 0.5 }, // SMALL - Suicide Pits (least visible)
      led: { base: 5, variation: 1 }, // MEDIUM - Blue LED
      training: { base: 9, variation: 1.5 }, // LARGE - Staff Training (important)
      helpline: { base: 5, variation: 1 }, // MEDIUM - Helpline
    };

  // Multi-centroid layout: cluster per country for clearer structure - now supports all 9 cities
  function getCountryCenters(w: number, h: number) {
    const padX = 80;
    const padY = 60;
    const innerW = w - padX * 2;
    const innerH = h - padY * 2;
    const columns = 3; // Increased to 3 columns for 9 cities
    const rows = 3; // 3 rows for 9 cities
    const cellW = innerW / columns;
    const cellH = innerH / rows;
    // Order all 9 cities in a logical layout
    const order: string[] = [
      'Singapore',
      'Tokyo',
      'Seoul', // Top row
      'London',
      'Paris',
      'Berlin', // Middle row
      'Toronto',
      'Amsterdam',
      'Singapore', // Bottom row (Singapore appears twice - will be handled)
    ];
    const centers: Record<
      string,
      { x: number; y: number; col: number; row: number }
    > = {} as any;

    // Track unique cities
    const uniqueCities = [...new Set(order)];

    uniqueCities.forEach((city, i) => {
      const row = Math.floor(i / columns);
      const col = i % columns;
      const cx = padX + col * cellW + cellW / 2;
      const cy = padY + row * cellH + cellH / 2;
      centers[city] = { x: cx, y: cy, col, row };
    });
    return centers;
  }
  let COUNTRY_CENTERS = getCountryCenters(width, height);

  // Estimate label bounding boxes for simple collision avoidance with dots
  function computeLabelRects(): Array<{
    city: string;
    x1: number;
    y1: number;
    x2: number;
    y2: number;
  }> {
    const rects: Array<{
      city: string;
      x1: number;
      y1: number;
      x2: number;
      y2: number;
    }> = [];
    const pad = 10; // extra safety around text
    Object.keys(COUNTRY_NAMES).forEach((city) => {
      const center = COUNTRY_CENTERS[city];
      const label = COUNTRY_NAMES[city];
      // crude width estimate: ~6.8px per character at 12px font
      const w = Math.max(60, label.length * 6.8);
      const h = 16; // font-size 12px -> bbox ~16px high
      const cx = center.x;
      const cy = center.y - 50; // keep in sync with render offset below
      rects.push({
        city: city,
        x1: cx - w / 2 - pad,
        y1: cy - h / 2 - pad,
        x2: cx + w / 2 + pad,
        y2: cy + h / 2 + pad,
      });
    });
    return rects;
  }

  const sceneNames: Record<string, string> = {
    'step-18.1': 'Genel Bakış',
    'step-18.2': 'Singapur',
    'step-18.3': 'Tokyo',
    'step-18.4': 'Londra',
    'step-18.5': 'Seul',
    'step-18.6': 'Paris',
    'step-18.7': 'Toronto',
    'step-18.8': 'Berlin',
    'step-18.9': 'Amsterdam',
  };

  type SceneKey = keyof typeof sceneNames;

  function resolveSceneKey(id: string | undefined | null): SceneKey {
    if (!id) return 'step-18.1';
    if (id && id in sceneNames) return id as SceneKey;
    const remapped = id.replace(/^step-18-(\d)$/, 'step-18.$1');
    if (remapped in sceneNames) return remapped as SceneKey;
    return 'step-18.1';
  }

  // Prepare bubbles for transition to European chart
  function prepareTransitionBubbles(): BubbleNode[] {
    return nodes.map((node) => ({
      id: `solution-${node.id}`,
      x: node.x,
      y: node.y,
      radius: node.radius,
      color: COUNTRY_COLORS[node.city] || '#95a5a6',
      type: 'solution' as const,
      data: { measureType: node.type, label: COUNTRY_NAMES[node.city] },
    }));
  }

  // Handle transition to step 19 - updated for new last step
  $: if (stepId === 'step-18.9') {
    // Prepare for transition when reaching the last step
    const bubbles = prepareTransitionBubbles();
    console.log(
      '🎬 GlobalSolutionsSwarm preparing transition bubbles:',
      bubbles.length
    );
    chartTransition.update((state) => ({
      ...state,
      bubbles: bubbles, // Replace instead of append to avoid duplicates
    }));
  }

  function buildNodesFromData(rows: GlobalSolutionRow[]): Node[] {
    // Now include all 9 cities instead of filtering to just 4
    const allowedCities = [
      'Singapore',
      'Tokyo',
      'London',
      'Seoul',
      'Paris',
      'Toronto',
      'Berlin',
      'Amsterdam',
    ];
    const filtered = rows.filter((row) => allowedCities.includes(row.city));

    // Expand each row into many micro-nodes to create a dense swarm
    // Base nodes per row; scaled by absolute change (bigger improvement → more nodes)
    const BASE_PER_ROW = 20; // Slightly reduced since we have more cities
    const nodesOut: Node[] = [];
    let nid = 0;
    filtered.forEach((r) => {
      const mag = Math.min(100, Math.abs(Number(r.change) || 0));
      const scale = 0.8 + mag / 100; // 0.8–1.8
      const count = Math.max(6, Math.round(BASE_PER_ROW * scale)); // Reduced minimum
      for (let k = 0; k < count; k++) {
        nodesOut.push({
          id: nid++,
          type: r.type as MeasureType,
          city: r.city,
          x: Math.random() * width,
          y: Math.random() * height,
          radius: calculateNodeRadius(
            r.type as MeasureType,
            Math.abs(Number(r.change) || 0)
          ),
          effectiveness: Math.abs(Number(r.change) || 0),
          year: Number(r.year) || 2000,
          station: r.station || 'Unknown',
        });
      }
    });
    return nodesOut;
  }

  function calculateNodeRadius(
    type: MeasureType,
    effectiveness?: number
  ): number {
    const config = SIZE_CONFIG[type];

    // Base size from type
    let baseSize = config.base;

    // Scale by effectiveness if provided (more effective = larger)
    if (effectiveness !== undefined) {
      baseSize = baseSize * (0.7 + effectiveness / 100); // 0.7x to 1.7x based on effectiveness
    }

    // Add some randomness
    const variation = config.variation;
    const finalSize = baseSize + (Math.random() - 0.5) * variation;

    return Math.max(1, finalSize);
  }

  function assignData(sceneKey: SceneKey) {
    nodes = buildNodesFromData(data);
  }

  function setupSimulation() {
    simulation = d3
      .forceSimulation<Node>(nodes)
      .force(
        'x',
        d3
          .forceX<Node>((d) => COUNTRY_CENTERS[d.city]?.x || width / 2)
          .strength(0.2)
      )
      .force(
        'y',
        d3
          .forceY<Node>((d) => (COUNTRY_CENTERS[d.city]?.y || height / 2) + 8)
          .strength(0.22)
      )
      .force(
        'collide',
        d3.forceCollide<Node>((d) => d.radius + 0.8).iterations(2)
      )
      // Soft push away from label rectangles so dots don't overlap titles
      .force('label-avoid', labelAvoidForce(0.4))
      .alphaDecay(0.03)
      .velocityDecay(0.2)
      .on('tick', tick);
  }

  // Custom force: gently push nodes out of label rectangles
  function labelAvoidForce(strength = 0.4) {
    return function (alpha: number) {
      if (!labelRects.length) return;
      for (const node of nodes) {
        const rect = labelRects.find((r) => r.city === node.city);
        if (!rect) continue;
        const nx = node.x;
        const ny = node.y;
        if (nx >= rect.x1 && nx <= rect.x2 && ny >= rect.y1 && ny <= rect.y2) {
          // push downwards out of the label area (labels sit above clusters)
          const dyBelow = rect.y2 - ny + 1;
          node.vy = (node.vy || 0) + (dyBelow * strength * alpha) / 8;
        }
      }
    } as unknown as d3.Force<Node, undefined>;
  }

  function tick() {
    if (!svg) return;

    svg
      .selectAll<SVGCircleElement, Node>('circle.dot')
      .attr('cx', (d) => d.x)
      .attr('cy', (d) => d.y)
      .attr('r', (d) => d.radius)
      .attr('fill', (d) => COUNTRY_COLORS[d.city] || '#95a5a6')
      .attr('opacity', (d) => {
        if (highlightedCity && d.city !== highlightedCity) return 0.3;
        return 0.85;
      });
  }

  function renderSVG() {
    if (!svgEl) return;

    svg = d3.select(svgEl);
    svg.selectAll('*').remove(); // Clear

    // Set viewBox for responsive scaling
    svg
      .attr('viewBox', `0 0 ${width} ${height}`)
      .attr('preserveAspectRatio', 'xMidYMid meet');

    // Background (transparent)
    svg
      .append('rect')
      .attr('width', width)
      .attr('height', height)
      .attr('fill', 'transparent')
      .attr('class', 'bg');

    // Recompute centers on render for current size
    COUNTRY_CENTERS = getCountryCenters(width, height);
    labelRects = computeLabelRects();

    // Title
    const currentSceneName = sceneNames[resolveSceneKey(stepId)];
    svg
      .append('text')
      .attr('x', 20)
      .attr('y', 28)
      .attr('class', 'title')
      .attr('fill', '#2c3e50')
      .attr('font-size', '16px')
      .attr('font-weight', '600')
      .text(`${currentSceneName} – Küresel Önlemler`);

    // Group labels (country names per cluster)
    const labels = svg.append('g').attr('class', 'group-labels');
    Object.keys(COUNTRY_NAMES).forEach((city) => {
      const c = COUNTRY_CENTERS[city];
      if (!c) return;
      labels
        .append('text')
        .attr('x', c.x)
        .attr('y', c.y - 50)
        .attr('text-anchor', 'middle')
        .attr('fill', COUNTRY_COLORS[city] || '#5b6770')
        .attr('font-size', '12px')
        .attr('font-weight', '600')
        .text(COUNTRY_NAMES[city]);
    });

    // Circles (data join)
    svg
      .selectAll('circle.dot')
      .data(nodes, (d: any) => d.id)
      .enter()
      .append('circle')
      .attr('class', 'dot')
      .attr('r', (d) => d.radius)
      .attr('cx', (d) => d.x)
      .attr('cy', (d) => d.y)
      .attr('fill', (d) => COUNTRY_COLORS[d.city] || '#95a5a6')
      .attr('opacity', (d) => {
        if (highlightedCity && d.city !== highlightedCity) return 0.3;
        return 0.85;
      })
      .style('transition', 'fill 0.3s ease, opacity 0.3s ease')
      .on('mouseover', function (event, d) {
        hoveredNode = d;
        mouseX = event.pageX;
        mouseY = event.pageY;
        // Highlight this node
        d3.select(this)
          .attr('opacity', 1)
          .attr('r', d.radius * 1.3);
      })
      .on('mouseout', function (event, d) {
        hoveredNode = null;
        // Restore opacity
        const opacity =
          highlightedCity && d.city !== highlightedCity ? 0.3 : 0.85;
        d3.select(this).attr('opacity', opacity).attr('r', d.radius);
      })
      .on('mousemove', function (event) {
        mouseX = event.pageX;
        mouseY = event.pageY;
      });

    // Effectiveness legend
    const legend = svg.append('g').attr('class', 'effectiveness-legend');
    legend
      .append('text')
      .attr('x', width - 120)
      .attr('y', 20)
      .attr('font-size', '12px')
      .attr('font-weight', '600')
      .attr('fill', '#2c3e50')
      .text('Etkinlik:');

    // Legend items
    const legendItems = [
      { color: '#e74c3c', label: 'Yüksek (80%+)', range: '80-100%' },
      { color: '#f39c12', label: 'Orta (40-79%)', range: '40-79%' },
      { color: '#95a5a6', label: 'Düşük (<40%)', range: '<40%' },
    ];

    legendItems.forEach((item, i) => {
      const itemGroup = legend
        .append('g')
        .attr('transform', `translate(${width - 120}, ${35 + i * 15})`);

      itemGroup.append('circle').attr('r', 4).attr('fill', item.color);

      itemGroup
        .append('text')
        .attr('x', 10)
        .attr('y', 2)
        .attr('font-size', '10px')
        .attr('fill', '#2c3e50')
        .text(item.range);
    });
  }

  function updateScene() {
    const sceneKey = resolveSceneKey(stepId);
    console.log('updateScene called:', {
      stepId,
      sceneKey,
      cityFilter,
      highlightedCity,
    });
    assignData(sceneKey);

    // Update title
    if (svg) {
      const currentSceneName = sceneNames[sceneKey];
      svg.select('text.title').text(`${currentSceneName} – Küresel Önlemler`);
    }

    // Restart simulation with energy
    if (simulation) {
      simulation.alpha(0.5).restart();
    }
  }

  // React to stepId changes
  $: if (stepId && simulation) {
    updateScene();
  }

  // React to highlightedCity changes for opacity only
  $: if (highlightedCity !== undefined && svg) {
    // Update opacity of existing circles
    svg.selectAll('circle.dot').attr('opacity', (d: any) => {
      if (highlightedCity && d.city !== highlightedCity) return 0.3;
      return 0.85;
    });
  }

  onMount(() => {
    console.log('GlobalSolutionsSwarm onMount:', {
      dataLength: data.length,
      stepId,
      cityFilter,
      highlightedCity,
    });
    nodes = buildNodesFromData(data);
    console.log('Initial nodes generated:', nodes.length);
    setupSimulation();
    renderSVG();
  });

  onDestroy(() => {
    if (simulation) {
      simulation.stop();
    }
  });
</script>

<div class="swarm-wrap" style="width: {width}px; height: {height}px;">
  <svg bind:this={svgEl} class="swarm-svg"></svg>

  <!-- Hover tooltip -->
  {#if hoveredNode}
    <div
      class="hover-tooltip"
      style="left: {mouseX + 10}px; top: {mouseY - 10}px;"
    >
      <div class="tooltip-header">
        <strong>{COUNTRY_NAMES[hoveredNode.city]}</strong>
        <span class="effectiveness-badge"
          >{hoveredNode.effectiveness}% azalma</span
        >
      </div>
      <div class="tooltip-content">
        <div class="station-info">
          {hoveredNode.station} ({hoveredNode.year})
        </div>
        <div class="measure-type">
          {#if hoveredNode.type === 'door'}
            Platform Kapısı
          {:else if hoveredNode.type === 'pit'}
            Ray Altı Çukur
          {:else if hoveredNode.type === 'led'}
            Mavi LED Işıklar
          {:else if hoveredNode.type === 'training'}
            Personel Eğitimi & AI
          {:else if hoveredNode.type === 'helpline'}
            Yardım Hattı
          {/if}
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  .swarm-wrap {
    width: 100%;
    display: block;
    position: relative;
    overflow: hidden;
    border-radius: 0;
    box-shadow: none;
    background: transparent;
  }

  .swarm-svg {
    width: 100%;
    height: 100%;
    display: block;
  }

  :global(.swarm-svg circle.dot) {
    cursor: pointer;
    transition:
      r 0.2s ease,
      opacity 0.2s ease;
  }

  :global(.swarm-svg circle.dot:hover) {
    r: calc(var(--base-radius, 4px) * 1.5);
    opacity: 1;
  }

  .hover-tooltip {
    position: fixed;
    background: rgba(255, 255, 255, 0.95);
    border: 1px solid #ddd;
    border-radius: 6px;
    padding: 8px 12px;
    font-size: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    pointer-events: none;
    z-index: 1000;
    max-width: 200px;
  }

  .tooltip-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 4px;
  }

  .effectiveness-badge {
    background: #e74c3c;
    color: white;
    padding: 2px 6px;
    border-radius: 3px;
    font-size: 10px;
    font-weight: 600;
  }

  .tooltip-content {
    color: #666;
  }

  .station-info {
    font-size: 11px;
    margin-bottom: 2px;
  }

  .measure-type {
    font-weight: 600;
    color: #2c3e50;
  }
</style>
