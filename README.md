<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mermaid + ELK Layout Example (Skypack CDN)</title>
  <style>
    .mermaid-container {
      max-width: 800px;
      margin: 32px auto;
      border: 1px solid #bbb;
      border-radius: 8px;
      background: #f8f9fa;
      overflow: auto;
      padding: 20px;
      box-sizing: border-box;
      height: 500px;
    }
    .mermaid svg {
      display: block;
      margin: auto;
      transition: transform 0.2s;
      transform-origin: center center;
    }
  </style>
  <script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    import elkLayouts from 'https://cdn.skypack.dev/@mermaid-js/layout-elk';

    mermaid.registerLayoutLoaders(elkLayouts);

    mermaid.initialize({
      layout: 'elk',
      theme: 'default'
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
