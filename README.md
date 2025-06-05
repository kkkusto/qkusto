// Register ELK layout
mermaid.registerLayoutLoaders(elkLayouts);

mermaid.initialize({
  layout: 'elk',
  theme: 'default',
  maxEdges: 8000, // Increase this as needed for larger graphs
});

// Zoom and scroll handler
function handleScroll(event) {
  event.preventDefault();

  const mermaidContainer = document.querySelector('.mermaid-container');
  const mermaidElement = document.querySelector('.mermaid svg');
  if (!mermaidContainer || !mermaidElement) return;

  const ZOOM_FACTOR = 0.2;
  const MIN_SCALE = 0.2;
  const MAX_SCALE = 10;

  if (event.ctrlKey) {
    let scale = parseFloat(mermaidElement.getAttribute('data-scale')) || 1;

    // Calculate zoom based on scroll direction
    if (event.deltaY < 0) {
      scale = Math.min(scale + ZOOM_FACTOR, MAX_SCALE);
    } else {
      scale = Math.max(scale - ZOOM_FACTOR, MIN_SCALE);
    }

    // Calculate mouse position relative to the SVG
    const rect = mermaidElement.getBoundingClientRect();
    const offsetX = event.clientX - rect.left;
    const offsetY = event.clientY - rect.top;

    // Convert to percentage for transform-origin
    const originX = (offsetX / rect.width) * 100;
    const originY = (offsetY / rect.height) * 100;

    // Apply scaling with transform-origin at mouse position
    requestAnimationFrame(() => {
      mermaidElement.style.transformOrigin = `${originX}% ${originY}%`;
      mermaidElement.style.transform = `scale(${scale})`;
      mermaidElement.setAttribute('data-scale', scale.toFixed(2));
    });
  }
  // Horizontal scroll with Shift
  else if (event.shiftKey) {
    mermaidContainer.scrollLeft += event.deltaY;
  }
  // Vertical scroll
  else {
    mermaidContainer.scrollTop += event.deltaY;
  }
}

// Attach event listener
document.addEventListener('wheel', handleScroll, { passive: false });
