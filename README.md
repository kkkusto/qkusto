<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  import elkLayouts from 'https://cdn.jsdelivr.net/npm/@mermaid-js/layout-elk@11/dist/mermaid-layout-elk.esm.min.mjs';

  // Register the ELK layout engine
  mermaid.registerLayoutLoaders(elkLayouts);

  // Initialize Mermaid with the ELK layout
  mermaid.initialize({
    layout: 'elk',
    theme: 'default',
    elk: {
      // Optional ELK-specific configurations
      mergeEdges: true,
      nodePlacementStrategy: 'NETWORK_SIMPLEX'
    }
  });
</script>
