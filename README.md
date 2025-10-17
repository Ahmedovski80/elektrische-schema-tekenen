elektrische-schema-app/
├── public/
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── components/
│   ├── App.js
│   ├── App.css
│   └── index.js
├── package.json
└── README.md
<!DOCTYPE html>
<html>
<head>
    <title>Elektrische Schema App</title>
    <style>
        .toolbar { padding: 10px; background: #f0f0f0; }
        .canvas { border: 1px solid #000; margin: 20px; }
        button { margin: 5px; padding: 10px; }
    </style>
</head>
<body>
    <h1>Elektrische Schema Tekenaar</h1>
    <div class="toolbar">
        <button onclick="addComponent('socket')">🔌 Stopcontact</button>
        <button onclick="addComponent('switch')">⚡ Schakelaar</button>
        <button onclick="addComponent('light')">💡 Lamp</button>
    </div>
    <canvas id="schemaCanvas" width="800" height="600" class="canvas"></canvas>
    
    <script>
        function addComponent(type) {
            alert(`Component ${type} toegevoegd!`);
            // Hier komt teken code
        }
    </script>
</body>
</html>
