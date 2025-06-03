function handleScroll(event) {
    event.preventDefault();
    const mermaidContainer = document.querySelector('.mermaid-container');
    // Select the SVG directly for scaling
    const mermaidElement = document.querySelector('.mermaid svg'); 
    if (!mermaidElement) return;

    if (event.ctrlKey) {
        let scale = Number(mermaidElement.getAttribute('data-scale')) || 1;
        const zoomFactor = 0.1;

        if (event.deltaY < 0) {
            scale += zoomFactor;
        } else {
            scale -= zoomFactor;
            if (scale < zoomFactor) scale = zoomFactor;
        }
        mermaidElement.style.transform = `scale(${scale})`;
        mermaidElement.setAttribute('data-scale', scale);
        // Keep transform-origin centered
        mermaidElement.style.transformOrigin = "center center";
    } else if (event.shiftKey) {
        // Horizontal scroll
        mermaidContainer.scrollLeft += event.deltaY;
    } else {
        // Vertical scroll
        mermaidContainer.scrollTop += event.deltaY;
    }
}

document.addEventListener('wheel', handleScroll, { passive: false });




<div class="mermaid-container" style="overflow: auto; width: 100%; max-width: 800px; height: 500px;">
  <div class="mermaid">
    flowchart TD
      A --> B
      B --> C
      C --> D
  </div>
</div>
