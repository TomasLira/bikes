<script>
    import * as d3 from "d3";
    export let lines=[];
    export let width=null;
    const firstColumnWidth = 150;
    const fileInfoMargin = 100;
    const dotsColumnX = firstColumnWidth + fileInfoMargin;
    const approxDotWidth = 10;
    const linesPerDot = 1;
    const baseY = 10;
    const totalLinesOffset = 20;
    const fileInfoHeight = baseY + totalLinesOffset;
    const dotRowHeight = 20;
    
    let colorScale = d3.scaleOrdinal(d3.schemeTableau10);
    
    
    function generateDots(file, svgWidth) {
        const totalDots = Math.ceil(file.lines.length / linesPerDot);
        const availableWidth = svgWidth - dotsColumnX;
        const maxDotsPerRow = Math.floor(availableWidth / approxDotWidth) || totalDots;
        let tspans = "";
        const dotRows = Math.ceil(totalDots / maxDotsPerRow);
        for (let r = 0; r < dotRows; r++) {
            const rowLines = file.lines.slice(r * maxDotsPerRow, (r + 1) * maxDotsPerRow);
            const rowDots = rowLines
            .map(line => `<tspan class="dot" style="fill:${colorScale(line.type)}">•</tspan>`)
            .join('');
            tspans += `<tspan x="${dotsColumnX}" dy="${r === 0 ? 0 : dotRowHeight + 'px'}">${rowDots}</tspan>`;
        }
        return tspans;
    }
    $: filesWithHeights = files.map(file => {
      const totalDots = Math.ceil(file.lines.length / linesPerDot);
      const availableWidth = width - dotsColumnX;
      const maxDotsPerRow = Math.floor(availableWidth / approxDotWidth) || totalDots;
      const dotRows = Math.ceil(totalDots / maxDotsPerRow);
      return { ...file, groupHeight: fileInfoHeight + dotRows * dotRowHeight };
    });
    $: positions = (() => {
      let pos = [], y = 0;
      for (const f of filesWithHeights) {
        pos.push(y);
        y += f.groupHeight;
      }
      return pos;
    })();
    
    
    let files = [];
    $: files = d3.groups(lines, d => d.file)
                .map(([name, lines]) => ({ name, lines }))
                .sort((a, b) => b.lines.length - a.lines.length);
    
    let svg;
    $: if (svg) {
        const svgWidth = 1200;
        const totalHeight = positions.length
            ? positions[positions.length - 1] + filesWithHeights[filesWithHeights.length - 1].groupHeight
            : 0;
        d3.select(svg)
            .attr('width', svgWidth)
            .attr('height', totalHeight)
            .style('overflow', 'hidden');
    
    
        const groups = d3.select(svg)
            .selectAll('g.file')
            .data(files, d => d.name);
        
        groups.exit().remove();
    
        const enterGroups = groups.enter()
            .append('g')
            .attr('class', 'file')
            .attr('transform', (d, i) => `translate(0, ${positions[i]})`);
        
        enterGroups.append('text')
            .attr('class', 'filename')
            .attr('x', 10)
            .attr('y', 5)
            .attr('dominant-baseline', 'hanging')
            .text(d => d.name);
    
        enterGroups.append('text')
            .attr('class', 'linecount')
            .attr('x', 10) 
            .attr('y', 20)
            .attr('dominant-baseline', 'hanging')
            .text(d => `${d.lines.length} lines`);
    
        //groups.attr('transform', (d, i) => `translate(0, ${positions[i]})`)
        //    .select('text.filename')
        //    .text(d => d.name);
    
        groups.transition()
            .duration(function(d, i) {
                const currentTransform = this.getAttribute("transform") || "translate(0,0)";
                const match = currentTransform.match(/translate\(\s*0\s*,\s*([0-9.]+)\s*\)/);
                const oldY = match ? +match[1] : 0;
                const newY = positions[i];
                const distance = Math.abs(newY - oldY);
                return distance * 2;
            })
            .attr('transform', (d, i) => `translate(0, ${positions[i]})`)
    
        groups.select('text.filename')
            .text(d => d.name);
    
        groups.select('text.linecount')
            .text(d => `${d.lines.length} lines`)
            .attr('x', 10);
        
        enterGroups.append('text')
            .attr('class', 'unit-dots')
            .attr('x', dotsColumnX)
            .attr('y', baseY - 2)
            .attr('dominant-baseline', 'mathematical')
            .attr('fill', "#1f77b4")
            .html(d => generateDots(d, svgWidth));
    
        groups.select('text.unit-dots')
            .html(d => generateDots(d, svgWidth))
            .attr('x', dotsColumnX)
            .attr('fill', "#1f77b4");
    
    }
    
    </script>    
    
    <svg bind:this={svg}></svg>
    
    <style>
    :global(text.filename){
        fill: light-dark(black, white);
    }
    :global(text.linecount){
        fill: grey;
    }
    </style>