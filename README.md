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

  // Handle zoom (Ctrl + scroll)
  if (event.ctrlKey) {
    let scale = parseFloat(mermaidElement.getAttribute('data-scale')) || 1;

    if (event.deltaY < 0) {
      scale = Math.min(scale + ZOOM_FACTOR, MAX_SCALE);
    } else {
      scale = Math.max(scale - ZOOM_FACTOR, MIN_SCALE);
    }

    // Apply scaling
    requestAnimationFrame(() => {
      mermaidElement.style.transform = `scale(${scale})`;
      mermaidElement.setAttribute('data-scale', scale.toFixed(2));
      mermaidElement.style.transformOrigin = 'center center';
    });
  }
  // Handle horizontal scroll (Shift + scroll)
  else if (event.shiftKey) {
    mermaidContainer.scrollLeft += event.deltaY;
  }
  // Handle vertical scroll
  else {
    mermaidContainer.scrollTop += event.deltaY;
  }
}

// Attach event listener
document.addEventListener('wheel', handleScroll, { passive: false });
