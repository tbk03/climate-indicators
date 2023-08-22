<script>
    export let x1;
    export let x2;
    export let y1;
    export let y2;
    export let arrowHeadWidth = 30;
    export let arrowHeadHeight = 30;
</script>

<defs>
  <marker
    id="arrowhead"
    markerWidth={arrowHeadWidth}
    markerHeight={arrowHeadHeight}
    refX="0"
    refY={arrowHeadHeight / 2}
    orient="auto"
  >
    <polygon points="0 0, {arrowHeadWidth} {arrowHeadHeight / 2}, 0 {arrowHeadHeight}" />
  </marker>
</defs>
<line
  {x1}
  y1={y1 - (arrowHeadHeight*0.2)}
  {x2}
  y2={y2 + (arrowHeadHeight*1.2)}
  stroke="#000"
  marker-end="url(#arrowhead)"
/>
