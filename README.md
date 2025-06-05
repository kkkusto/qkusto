<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mermaid Diagram with Region-based Zoom (Working)</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
    }
    .mermaid-container {
      width: 100vw;
      height: 100vh;
      overflow: auto;
      background: #f8f9fa;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }
    .mermaid svg {
      display: block;
      margin: auto;
      transition: transform 0.1s;
    }
  </style>
  <script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    import elkLayouts from 'https://cdn.skypack.dev/@mermaid-js/layout-elk';

    // Register ELK layouts
    mermaid.registerLayoutLoaders(elkLayouts);

    // Initialize Mermaid
    mermaid.initialize({
      layout: 'elk',
      theme: 'default',
      maxEdges: 8000
    });

    // Wait for DOM content loaded
    window.addEventListener('DOMContentLoaded', () => {
      const mermaidContainer = document.querySelector('.mermaid-container');
      const diagram = document.querySelector('.mermaid');

      // Render Mermaid diagram explicitly
      mermaid.init(undefined, diagram).then(() => {
        // Now the SVG is ready
        const mermaidElement = diagram.querySelector('svg');
        if (!mermaidElement) {
          console.error('Mermaid SVG not found.');
          return;
        }

        document.addEventListener('wheel', (event) => {
          if (!mermaidElement) return;

          const ZOOM_FACTOR = 0.2;
          const MIN_SCALE = 0.2;
          const MAX_SCALE = 10;

          if (event.ctrlKey) {
            event.preventDefault();

            let scale = parseFloat(mermaidElement.getAttribute('data-scale')) || 1;
            scale += (event.deltaY < 0 ? 1 : -1) * ZOOM_FACTOR;
            scale = Math.max(MIN_SCALE, Math.min(scale, MAX_SCALE));

            const rect = mermaidElement.getBoundingClientRect();
            const offsetX = event.clientX - rect.left;
            const offsetY = event.clientY - rect.top;
            const originX = (offsetX / rect.width) * 100;
            const originY = (offsetY / rect.height) * 100;

            requestAnimationFrame(() => {
              mermaidElement.style.transformOrigin = `${originX}% ${originY}%`;
              mermaidElement.style.transform = `scale(${scale})`;
              mermaidElement.setAttribute('data-scale', scale.toFixed(2));
            });
          }
          else if (event.shiftKey) {
            mermaidContainer.scrollLeft += event.deltaY;
          }
          else {
            mermaidContainer.scrollTop += event.deltaY;
          }
        }, { passive: false });
      });
    });
  </script>
</head>
<body>
  <div class="mermaid-container">
    <div class="mermaid">
flowchart TD
  Start((Start)) --> A[Do something]
  A --> B{Decision?}
  B -- Yes --> C[Keep going]
  B -- No  --> D[Stop]
  C --> E[Final Step]
  D --> E
  E((End))
    </div>
  </div>
</body>
</html>
