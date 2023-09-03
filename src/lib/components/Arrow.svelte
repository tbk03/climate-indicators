<script>
    export let x1;
    export let x2;
    export let y1;
    export let y2;
    export let arrowHeadWidth = 30;
    export let arrowHeadHeight = 30;
    export let color = '#757373';

    let strokeWidth = 3; 
    let arrowHeadWidthAdj = arrowHeadWidth - (2 * strokeWidth);
    let arrowHeadHeightAdj = arrowHeadHeight - (2 * strokeWidth);

</script>

<defs>
  <marker
    id="arrowhead"
    markerWidth={arrowHeadWidthAdj}
    markerHeight={arrowHeadHeightAdj}
    refX="0"
    refY={arrowHeadHeightAdj / 2}
    orient="auto"
    fill={color}
  >
    <polygon points="0 0, {arrowHeadWidthAdj} {arrowHeadHeightAdj / 2}, 0 {arrowHeadHeightAdj}" />
  </marker>
</defs>
<line
  {x1}
  y1={y1 - (arrowHeadHeight*0.2)}
  {x2}
  y2={y2 + (arrowHeadHeight*1.2)}
  stroke={color}
  stroke-width={strokeWidth}
  marker-end="url(#arrowhead)"
/>
