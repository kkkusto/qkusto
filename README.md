<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mermaid Diagram: Zoom + Pan</title>
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
      position: relative;
      cursor: grab;
    }
    .mermaid-container:active {
      cursor: grabbing;
    }
    .mermaid svg {
      display: block;
      margin: auto;
      transition: transform 0.1s;
      /* Initially no transform-origin set; we set it dynamically */
    }
  </style>
  <script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    import elkLayouts from 'https://cdn.skypack.dev/@mermaid-js/layout-elk';

    mermaid.registerLayoutLoaders(elkLayouts);

    mermaid.initialize({
      layout: 'elk',
      theme: 'default',
      maxEdges: 8000
    });

    window.addEventListener('DOMContentLoaded', () => {
      const mermaidContainer = document.querySelector('.mermaid-container');
      const diagram = document.querySelector('.mermaid');

      // Initial pan/zoom state
      let panX = 0;
      let panY = 0;
      let isPanning = false;
      let startX = 0;
      let startY = 0;

      // Render diagram
      mermaid.init(undefined, diagram).then(() => {
        const mermaidElement = diagram.querySelector('svg');
        if (!mermaidElement) {
          console.error('Mermaid SVG not found.');
          return;
        }

        // Initialize scale
        let scale = 1;
        mermaidElement.setAttribute('data-scale', scale.toFixed(2));
        updateTransform();

        // Zooming
        document.addEventListener('wheel', (event) => {
          if (!mermaidElement) return;

          const ZOOM_FACTOR = 0.2;
          const MIN_SCALE = 0.2;
          const MAX_SCALE = 10;

          if (event.ctrlKey) {
            event.preventDefault();

            // Zoom calculation
            let newScale = scale + (event.deltaY < 0 ? 1 : -1) * ZOOM_FACTOR;
            newScale = Math.max(MIN_SCALE, Math.min(newScale, MAX_SCALE));

            // Mouse position for zoom origin
            const rect = mermaidElement.getBoundingClientRect();
            const offsetX = event.clientX - rect.left;
            const offsetY = event.clientY - rect.top;
            const originX = (offsetX / rect.width) * 100;
            const originY = (offsetY / rect.height) * 100;

            // Adjust pan to zoom around mouse pointer
            const prevWidth = rect.width * scale;
            const prevHeight = rect.height * scale;
            const newWidth = rect.width * newScale;
            const newHeight = rect.height * newScale;
            panX = panX - ((offsetX - panX) / scale) * (newScale - scale);
            panY = panY - ((offsetY - panY) / scale) * (newScale - scale);

            // Apply transform
            scale = newScale;
            mermaidElement.setAttribute('data-scale', scale.toFixed(2));
            mermaidElement.style.transformOrigin = `${originX}% ${originY}%`;
            updateTransform();
          }
          else if (event.shiftKey) {
            mermaidContainer.scrollLeft += event.deltaY;
          }
          else {
            mermaidContainer.scrollTop += event.deltaY;
          }
        }, { passive: false });

        // Panning
        mermaidElement.addEventListener('mousedown', (event) => {
          if (event.button !== 0) return; // Only left mouse button
          isPanning = true;
          startX = event.clientX - panX;
          startY = event.clientY - panY;
          mermaidContainer.style.cursor = 'grabbing';
          event.preventDefault();
        });

        document.addEventListener('mousemove', (event) => {
          if (!isPanning) return;
          panX = event.clientX - startX;
          panY = event.clientY - startY;
          updateTransform();
        });

        document.addEventListener('mouseup', () => {
          if (isPanning) {
            isPanning = false;
            mermaidContainer.style.cursor = 'grab';
          }
        });

        function updateTransform() {
          mermaidElement.style.transform = `translate(${panX}px, ${panY}px) scale(${scale})`;
        }
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
