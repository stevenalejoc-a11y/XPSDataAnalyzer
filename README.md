[Analizador 17.txt](https://github.com/user-attachments/files/22504438/Analizador.17.txt)
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>XPS Data Analyzer Interactivo</title>
    <script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://unpkg.com/optimization-js@1.0.1/dist/optimization.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
            min-height: 100vh;
            padding: 20px;
            color: #333;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(90deg, #2c3e50, #3498db);
            color: white;
            padding: 20px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }
        
        .header p {
            font-size: 1.1rem;
            opacity: 0.9;
        }
        
        .content {
            display: flex;
            flex-wrap: wrap;
        }
        
        .sidebar {
            width: 300px;
            background: #f8f9fa;
            padding: 20px;
            border-right: 1px solid #dee2e6;
            overflow-y: auto;
            max-height: 80vh;
        }
        
        .main-content {
            flex: 1;
            padding: 20px;
        }
        
        .section {
            margin-bottom: 25px;
            background: white;
            border-radius: 10px;
            padding: 15px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        
        .section h3, .section h4 {
            color: #2c3e50;
            margin-bottom: 15px;
            font-size: 1.2rem;
            border-bottom: 2px solid #3498db;
            padding-bottom: 8px;
        }
        .section h4 {
            font-size: 1rem;
            border-bottom: 1px solid #bdc3c7;
            margin-top: 15px;
        }
        
        .btn {
            background: linear-gradient(90deg, #3498db, #2980b9);
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 0.95rem;
            margin: 5px 0;
            transition: all 0.3s ease;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
            width: 100%;
        }
        
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        }
        
        .btn:active {
            transform: translateY(0);
        }
        
        .btn-danger {
            background: linear-gradient(90deg, #e74c3c, #c0392b);
        }
        
        .btn-success {
            background: linear-gradient(90deg, #2ecc71, #27ae60);
        }
        
        .btn-warning {
            background: linear-gradient(90deg, #f39c12, #d35400);
        }

        .btn-info {
            background: linear-gradient(90deg, #5bc0de, #337ab7); /* New color for auto fit */
        }
        
        .file-upload-label {
            display: block;
            padding: 12px;
            background: linear-gradient(90deg, #9b59b6, #8e44ad);
            color: white;
            border-radius: 5px;
            text-align: center;
            font-weight: bold;
            transition: all 0.3s ease;
            cursor: pointer;
        }
        
        .file-upload-label:hover {
            background: linear-gradient(90deg, #8e44ad, #7d3c98);
        }
        input[type=file] { display: none; }
        
        .control-group {
            margin-bottom: 15px;
        }
        
        .control-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            color: #555;
            font-size: 0.9rem;
        }
        
        .slider {
            width: 100%;
            height: 6px;
            border-radius: 5px;
            background: #d3d3d3;
            outline: none;
            -webkit-appearance: none;
            cursor: pointer;
        }
        
        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3498db;
            cursor: pointer;
        }
        
        .slider::-moz-range-thumb {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3498db;
            cursor: pointer;
            border: none;
        }
        
        .chart-container {
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            height: 80vh;
        }
        
        .status-bar {
            background: #34495e;
            color: white;
            padding: 10px 20px;
            font-size: 0.9rem;
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 10px; /* Espacio entre elementos */
        }
        
        .notification {
            position: fixed; top: 20px; right: 20px; padding: 15px 20px;
            background: #2ecc71; color: white; border-radius: 5px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
            transform: translateX(400px); transition: transform 0.3s ease;
            z-index: 1000;
        }
        .notification.show { transform: translateX(0); }
        .notification.error { background: #e74c3c; }

        /* Style for fit data display */
        #fitDataOutput {
            max-height: 200px;
            overflow-y: auto;
            border: 1px solid #dee2e6;
            border-radius: 5px;
            padding: 10px;
            background-color: #f0f0f0;
            font-family: 'Courier New', Courier, monospace;
            font-size: 0.85rem;
            white-space: pre-wrap; /* Preserve whitespace and wrap long lines */
            margin-top: 10px;
        }
        #fitDataOutput b {
            color: #2c3e50;
        }
        #elementStatus {
            margin-top: 10px;
            font-weight: bold;
            color: #28a745; /* Green for success */
        }
        #elementStatus.error {
            color: #dc3545; /* Red for error/warning */
        }
        
        @media (max-width: 992px) {
            .content { flex-direction: column; }
            .sidebar { width: 100%; max-height: none; border-right: none; border-bottom: 1px solid #dee2e6;}
            .chart-container { height: 60vh; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>XPS Data Analyzer</h1>
            <p>Herramienta avanzada para análisis de espectroscopía de fotoelectrones (XPS)</p>
        </div>
        
        <div class="content">
            <div class="sidebar">
                <div class="section">
                    <h3>Cargar Datos</h3>
                    <label for="fileInput" class="file-upload-label">Seleccionar archivo</label>
                    <input type="file" id="fileInput" accept=".xlsx, .xls, .csv">
                    <button id="loadSampleBtn" class="btn">Cargar Datos de Ejemplo</button>
                    <div id="fileName" style="margin-top: 10px; font-style: italic; color: #7f8c9d; word-wrap: break-word;"></div>
                </div>
                
                <div class="section">
                    <h3>Controles</h3>
                    <button id="resetBtn" class="btn btn-danger">Restablecer Todo</button>
                    <button id="detectPeaksBtn" class="btn">Detectar Picos</button>
                    <button id="deconvolutionBtn" class="btn btn-warning">Deconvolución Manual Zr 3d</button>
                    <button id="autoFitBtn" class="btn btn-info">Ajuste Automático Zr 3d</button> <button id="exportBtn" class="btn btn-success">Exportar Gráfica</button>
                    <button id="exportFitDataBtn" class="btn btn-success">Exportar Datos de Ajuste</button> </div>
                
                <div class="section" id="deconvolution-controls" style="display: none;">
                    <h3>Parámetros de Ajuste Manual</h3>
                    <h4>Pico 1 (Zr 3d₅/₂)</h4>
                    <div class="control-group">
                        <label>Amplitud: <span id="p1_amp_val">1580</span></label>
                        <input type="range" id="p1_amp" min="0" max="2500" value="1580" class="slider">
                    </div>
                    <div class="control-group">
                        <label>Centro (eV): <span id="p1_cen_val">182.180</span></label>
                        <input type="range" id="p1_cen" min="175" max="190" value="182.18" step="0.001" class="slider">
                    </div>
                    <div class="control-group">
                        <label>FWHM (eV): <span id="p1_fwhm_val">1.68</span></label>
                        <input type="range" id="p1_fwhm" min="0.01" max="5" value="1.68" step="0.01" class="slider">
                    </div>

                    <h4>Pico 2 (Zr 3d₃/₂)</h4>
                    <div class="control-group">
                        <label>Amplitud: <span id="p2_amp_val">1015</span></label>
                        <input type="range" id="p2_amp" min="0" max="2000" value="1015" class="slider">
                    </div>
                    <div class="control-group">
                        <label>Centro (eV): <span id="p2_cen_val">184.560</span></label>
                        <input type="range" id="p2_cen" min="175" max="190" value="184.56" step="0.001" class="slider">
                    </div>
                    <div class="control-group">
                        <label>FWHM (eV): <span id="p2_fwhm_val">1.88</span></label>
                        <input type="range" id="p2_fwhm" min="0.01" max="5" value="1.88" step="0.01" class="slider">
                    </div>

                    <h4>Línea de Base (Lineal)</h4>
                    <div class="control-group">
                        <label>Inicio (Y): <span id="bg_start_val">25</span></label>
                        <input type="range" id="bg_start" min="-100" max="500" value="25" class="slider">
                    </div>
                    <div class="control-group">
                        <label>Fin (Y): <span id="bg_end_val">15</span></label>
                        <input type="range" id="bg_end" min="-100" max="500" value="15" step="1" class="slider">
                    </div>
                    <button id="resetParamsBtn" class="btn btn-danger">Restablecer Parámetros</button>
                </div>

                <div class="section" id="fit-data-section" style="display: none;">
                    <h3>Datos del Ajuste Total</h3>
                    <div id="fitDataOutput"></div>
                    <div id="elementStatus"></div> </div>
            </div>
            
            <div class="main-content">
                <div id="chart" class="chart-container"></div>
            </div>
        </div>
        
        <div class="status-bar">
            <div id="statusText">Listo para cargar datos</div>
            <div id="fitInfo"></div> 
            <div id="areaInfo"></div> <div id="dataInfo">Sin datos cargados</div>
        </div>
    </div>
    
    <div id="notification" class="notification"></div>

    <script>
        // Global state
        let rawData = []; 
        let smoothedData = []; // New variable for smoothed data
        let peaks = []; 
        let fitTraces = [];
        let totalFitData = []; // New global variable to store the total fit data
        const defaultFitParams = { // Default params, adjusted for broader range
            p1_amp: 1000, p1_cen: 182.0, p1_fwhm: 1.5,
            p2_amp: 600, p2_cen: 184.4, p2_fwhm: 1.5,
            bg_start: 10, bg_end: 5
        };

        // DOM Elements
        const fileInput = document.getElementById('fileInput');
        const loadSampleBtn = document.getElementById('loadSampleBtn');
        const fileName = document.getElementById('fileName');
        const resetBtn = document.getElementById('resetBtn');
        const detectPeaksBtn = document.getElementById('detectPeaksBtn');
        const deconvolutionBtn = document.getElementById('deconvolutionBtn');
        const autoFitBtn = document.getElementById('autoFitBtn'); // Nuevo DOM element
        const exportBtn = document.getElementById('exportBtn');
        const exportFitDataBtn = document.getElementById('exportFitDataBtn'); // New DOM element
        const statusText = document.getElementById('statusText');
        const dataInfo = document.getElementById('dataInfo');
        const fitInfo = document.getElementById('fitInfo');
        const areaInfo = document.getElementById('areaInfo'); // Nuevo DOM element
        const notification = document.getElementById('notification');
        const deconvolutionControls = document.getElementById('deconvolution-controls');
        const fitDataSection = document.getElementById('fit-data-section'); // New DOM element
        const fitDataOutput = document.getElementById('fitDataOutput');     // New DOM element
        const elementStatus = document.getElementById('elementStatus');     // New DOM element
        const resetParamsBtn = document.getElementById('resetParamsBtn');
        
        // Deconvolution parameter inputs and value displays
        const paramInputs = {};
        const paramValues = {};
        Object.keys(defaultFitParams).forEach(key => {
            paramInputs[key] = document.getElementById(key);
            paramValues[key] = document.getElementById(`${key}_val`);
        });

        // Event Listeners
        fileInput.addEventListener('change', handleFileUpload);
        loadSampleBtn.addEventListener('click', loadSampleData);
        resetBtn.addEventListener('click', resetAll);
        detectPeaksBtn.addEventListener('click', detectAndShowPeaks);
        deconvolutionBtn.addEventListener('click', toggleManualDeconvolution); // Cambiado a manual
        autoFitBtn.addEventListener('click', runAutomaticFit); // Nuevo event listener
        exportBtn.addEventListener('click', exportChart);
        exportFitDataBtn.addEventListener('click', exportTotalFitData); // New event listener
        resetParamsBtn.addEventListener('click', resetAndRefit);

        // Add listeners for all sliders to update plot and text
        Object.keys(paramInputs).forEach(key => {
            paramInputs[key].addEventListener('input', () => {
                // Al actualizar el valor, asegúrese de que se muestre con la misma precisión que el 'step'
                // O con una precisión fija para la visualización
                if (key.includes('cen')) {
                    paramValues[key].textContent = parseFloat(paramInputs[key].value).toFixed(3); // 3 decimales para 0.001
                } else if (key.includes('fwhm')) { 
                    paramValues[key].textContent = parseFloat(paramInputs[key].value).toFixed(2); // 2 decimales para 0.01
                } else {
                    paramValues[key].textContent = parseFloat(paramInputs[key].value).toFixed(0); // For amplitude/bg, no decimals
                }
                runFit(); 
            });
        });


        function showNotification(message, isError = false) {
            notification.textContent = message;
            notification.className = isError ? 'notification error show' : 'notification show';
            setTimeout(() => notification.classList.remove('show'), 3000);
        }

        // --- NEW: Simple Moving Average (SMA) filter function ---
        function applySmoothing(data, windowSize) {
            if (windowSize < 1) windowSize = 1; // Ensure windowSize is at least 1
            if (windowSize % 2 === 0) windowSize += 1; // Ensure odd window size for symmetry
            
            const smoothed = [];
            const halfWindow = Math.floor(windowSize / 2);

            for (let i = 0; i < data.length; i++) {
                let sum = 0;
                let count = 0;
                for (let j = -halfWindow; j <= halfWindow; j++) {
                    if (data[i + j]) { // Check if the index exists
                        sum += data[i + j].y;
                        count++;
                    }
                }
                smoothed.push({ x: data[i].x, y: sum / count });
            }
            return smoothed;
        }

        function loadSampleData() {
            resetAll();
            const startEnergy = 178, endEnergy = 188, step = 0.05;
            for (let energy = startEnergy; energy <= endEnergy; energy += step) {
                let y = gaussian(energy, 1550, 182.15, 1.7) + gaussian(energy, 980, 184.55, 1.9);
                y += 40 - (energy - startEnergy) * 2;
                y += (Math.random() - 0.5) * Math.sqrt(Math.abs(y)) * 0.5; // Added noise
                rawData.push({ x: parseFloat(energy.toFixed(2)), y: parseFloat(Math.max(0, y).toFixed(2)) });
            }
            
            // Apply smoothing after raw data is generated
            smoothedData = applySmoothing(rawData, 5); // Apply a window of 5 for smoothing
            
            fileName.textContent = "Datos de ejemplo (Zr 3d)";
            dataInfo.textContent = `${rawData.length} puntos cargados (Suavizado: ${smoothedData.length} puntos)`;
            statusText.textContent = 'Datos de ejemplo cargados y suavizados';
            showNotification('Datos de ejemplo cargados y suavizados');
            drawPlot();
        }

        function handleFileUpload(e) {
            const file = e.target.files[0];
            if (!file) return;
            resetAll();
            fileName.textContent = file.name;
            statusText.textContent = 'Procesando archivo...';
            
            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const data = new Uint8Array(event.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const worksheet = workbook.Sheets[workbook.SheetNames[0]];
                    let jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1, defval: ''});
                    
                    if (!jsonData || jsonData.length === 0) throw new Error('El archivo está vacío.');
                    
                    // Determinar el índice de inicio basado en si la primera fila es numérica o un encabezado.
                    // También se asegura de que reemplace las comas por puntos para el parsing.
                    let startIndex = 0;
                    if (jsonData.length > 0 && jsonData[0].length >= 2) {
                        const firstValX = String(jsonData[0][0]).replace(',', '.');
                        const firstValY = String(jsonData[0][1]).replace(',', '.');
                        if (isNaN(parseFloat(firstValX)) || isNaN(parseFloat(firstValY))) {
                            startIndex = 1; // Si la primera fila no es un número válido, asume que es un encabezado.
                        }
                    }
                    
                    for (let i = startIndex; i < jsonData.length; i++) {
                        const row = jsonData[i];
                        if (row && row.length >= 2) {
                            // Replace comma with period for decimal parsing
                            const x_str = String(row[0]).replace(',', '.');
                            const y_str = String(row[1]).replace(',', '.');

                            const x = parseFloat(x_str);
                            const y = parseFloat(y_str);
                            if (!isNaN(x) && !isNaN(y)) {
                                // Importante: Asegurarse de que las intensidades no sean negativas
                                rawData.push({x: parseFloat(x.toFixed(2)), y: parseFloat(Math.max(0, y).toFixed(2))});
                            }
                        }
                    }
                    
                    if (rawData.length === 0) throw new Error('No se encontraron datos numéricos válidos.');
                    
                    rawData.sort((a, b) => a.x - b.x); // Ensure data is sorted
                    
                    // Apply smoothing after raw data is loaded and cleaned
                    smoothedData = applySmoothing(rawData, 5); // Apply a window of 5 for smoothing
                    
                    dataInfo.textContent = `${rawData.length} puntos cargados (Suavizado: ${smoothedData.length} puntos)`;
                    statusText.textContent = 'Datos cargados y suavizados correctamente';
                    showNotification('Archivo cargado y suavizado exitosamente');
                    drawPlot(); // Dibuja el gráfico con los nuevos datos
                } catch (error) {
                    console.error('Error al procesar archivo:', error);
                    statusText.textContent = 'Error al procesar archivo';
                    showNotification('Error: ' + error.message, true);
                }
            };
            reader.readAsArrayBuffer(file);
        }

        function drawPlot() {
            // Now, use smoothedData for plotting if available, otherwise rawData
            const dataToPlot = smoothedData.length > 0 ? smoothedData : rawData;

            if (dataToPlot.length === 0) {
                Plotly.purge('chart');
                return;
            }
            
            const traces = [];
            traces.push({
                x: dataToPlot.map(d => d.x), y: dataToPlot.map(d => d.y),
                type: 'scatter', mode: 'lines', name: 'Datos Experimentales (Suavizados)', // Updated name
                line: {color: 'grey', width: 2},
                hovertemplate: 'Energía: %{x:.2f} eV<br>Intensidad: %{y:.2f}<extra></extra>' // Formato hover
            });

            if (peaks.length > 0) {
                traces.push({
                    x: peaks.map(p => p.x), y: peaks.map(p => p.y),
                    type: 'scatter', mode: 'markers', name: 'Picos Detectados',
                    marker: {color: '#2ecc71', size: 10, symbol: 'triangle-up'},
                    hovertemplate: 'Pico @ %{x:.2f} eV<br>Intensidad: %{y:.2f}<extra></extra>' // Formato hover
                });
            }

            // Add fit traces if they exist
            traces.push(
                { x: dataToPlot.map(d => d.x), y: dataToPlot.map((d, i) => gaussian(d.x, parseFloat(paramInputs.p1_amp.value), parseFloat(paramInputs.p1_cen.value), parseFloat(paramInputs.p1_fwhm.value))), mode: 'lines', name: 'Pico 1', line: { color: 'blue', dash: 'dash' }, hovertemplate: 'Pico 1 @ %{x:.2f} eV<br>Intensidad: %{y:.2f}<extra></extra>' },
                { x: dataToPlot.map(d => d.x), y: dataToPlot.map((d, i) => gaussian(d.x, parseFloat(paramInputs.p2_amp.value), parseFloat(paramInputs.p2_cen.value), parseFloat(paramInputs.p2_fwhm.value))), mode: 'lines', name: 'Pico 2', line: { color: 'green', dash: 'dash' }, hovertemplate: 'Pico 2 @ %{x:.2f} eV<br>Intensidad: %{y:.2f}<extra></extra>' },
                { x: dataToPlot.map(d => d.x), y: dataToPlot.map((d, i) => {
                    const params = {};
                    Object.keys(paramInputs).forEach(key => params[key] = parseFloat(paramInputs[key].value));
                    // Ensure calculation uses the x-values from dataToPlot
                    const progress = (dataToPlot[0].x - d.x) / (dataToPlot[0].x - dataToPlot[dataToPlot.length - 1].x); // Normalized progress across x-axis
                    const background_y_val = params.bg_start + progress * (params.bg_end - params.bg_start);
                    return gaussian(d.x, params.p1_amp, params.p1_cen, params.p1_fwhm) + gaussian(d.x, params.p2_amp, params.p2_cen, params.p2_fwhm) + background_y_val;
                }), mode: 'lines', name: 'Ajuste Total', line: { color: 'red', width: 2.5 }, hovertemplate: 'Ajuste Total @ %{x:.2f} eV<br>Intensidad: %{y:.2f}<extra></extra>' }
            );


            const layout = {
                title: 'Análisis XPS',
                xaxis: { 
                    title: 'Energía de Enlace (eV)', 
                    autorange: 'reversed', 
                    tickformat: '.2f' 
                },
                yaxis: { 
                    title: 'Intensidad (Cuentas)',
                    autorange: true, 
                    tickformat: '.2f' 
                },
                hovermode: 'closest', showlegend: true,
                legend: {orientation: 'h', y: -0.2, x: 0.5, xanchor: 'center'}
            };
            
            Plotly.react('chart', traces, layout, {responsive: true});
            statusText.textContent = 'Gráfica actualizada';
        }

        // --- DECONVOLUTION AND FITTING FUNCTIONS ---
        function gaussian(x, amp, cen, fwhm) {
            const sigma = fwhm / (2 * Math.sqrt(2 * Math.log(2)));
            // Smallest non-zero sigma to prevent division by zero or NaN
            if (sigma < 1e-9) return 0; // Prevent division by very small sigma
            return amp * Math.exp(-((x - cen) ** 2) / (2 * sigma ** 2));
        }

        // NUEVA FUNCIÓN: Calcular el área bajo una curva Gaussiana
        function gaussianArea(amp, fwhm) {
            const sigma = fwhm / (2 * Math.sqrt(2 * Math.log(2)));
            if (sigma < 1e-9) return 0; // Prevent division by very small sigma
            return amp * sigma * Math.sqrt(2 * Math.PI);
        }

        // Función de costo para el optimizador (suma de los cuadrados de los residuos)
        // El optimizador buscará minimizar este valor.
        function calculateSumOfSquaredResiduals(params) {
            const [p1_amp, p1_cen, p1_fwhm, p2_amp, p2_cen, p2_fwhm, bg_start, bg_end] = params;

            // Use smoothedData for fitting
            const xValues = smoothedData.map(d => d.x);
            const experimentalY = smoothedData.map(d => d.y);

            let sumOfSquares = 0;
            for (let i = 0; i < smoothedData.length; i++) { // Use smoothedData length
                const x = xValues[i];
                const expY = experimentalY[i];

                const peak1_val = gaussian(x, p1_amp, p1_cen, p1_fwhm);
                const peak2_val = gaussian(x, p2_amp, p2_cen, p2_fwhm);
                
                // Calcular la línea base para este punto x
                const progress = (xValues[0] - x) / (xValues[0] - xValues[xValues.length - 1]); // Use xValues for range
                const background_val = bg_start + progress * (bg_end - bg_start);
                
                const fittedY = peak1_val + peak2_val + background_val;
                
                sumOfSquares += (expY - fittedY) ** 2;
            }
            return sumOfSquares;
        }

        function calculateReducedChiSquare(experimentalData, fittedY, numParams) {
            const n = experimentalData.length; 
            if (n <= numParams) return Infinity; 
            
            let chi2 = 0;
            for (let i = 0; i < n; i++) {
                const expY = experimentalData[i].y;
                const fitY = fittedY[i];

                // Usamos el valor ajustado (esperado) como la varianza, asumiendo ruido de Poisson.
                // Usar Math.max(0.1, Math.abs(fitY)) para evitar divisiones por cero o valores muy pequeños.
                chi2 += ((expY - fitY) ** 2) / Math.max(0.1, Math.abs(fitY)); 
            }
            const degreesOfFreedom = n - numParams;
            return chi2 / degreesOfFreedom;
        }

        function toggleManualDeconvolution() {
            const isVisible = deconvolutionControls.style.display === 'block';
            if (isVisible) {
                deconvolutionControls.style.display = 'none';
                fitTraces = []; 
                fitInfo.textContent = '';
                areaInfo.textContent = ''; 
                fitDataSection.style.display = 'none'; 
                fitDataOutput.innerHTML = ''; 
                elementStatus.innerHTML = ''; // Clear element status
                drawPlot();
            } else {
                if (smoothedData.length === 0) { // Check smoothedData
                    showNotification('Cargue los datos primero para la deconvolución.', true);
                    return;
                }
                deconvolutionControls.style.display = 'block';
                fitDataSection.style.display = 'block'; 
                runFit();
            }
        }

        /**
         * runAutomaticFit()
         * Esta función ahora realiza un ajuste automático REAL utilizando optimization.js.
         */
        async function runAutomaticFit() {
            if (smoothedData.length === 0) { 
                showNotification('Cargue los datos primero para el ajuste automático.', true);
                return;
            }

            statusText.textContent = 'Iniciando ajuste automático...';
            showNotification('Ajuste automático en progreso...');
            
            deconvolutionControls.style.display = 'none'; 
            fitDataSection.style.display = 'block'; 
            fitDataOutput.innerHTML = ''; 
            elementStatus.innerHTML = ''; // Clear element status before new fit

            // Determinar el rango de energías y la intensidad máxima de los datos suavizados.
            const minEnergy = Math.min(...smoothedData.map(d => d.x));
            const maxEnergy = Math.max(...smoothedData.map(d => d.x));
            const maxIntensity = Math.max(...smoothedData.map(d => d.y));
            const minIntensity = Math.min(...smoothedData.map(d => d.y));


            const initialParams = [
                // Estos valores iniciales son estimaciones robustas basadas en los datos
                maxIntensity * 0.7,     // p1_amp
                (minEnergy + maxEnergy) / 2 - 1.2, // p1_cen (aproximadamente -1.2eV del centro del rango)
                1.5,                    // p1_fwhm
                maxIntensity * 0.4,     // p2_amp
                (minEnergy + maxEnergy) / 2 + 1.2, // p2_cen (aproximadamente +1.2eV del centro del rango)
                1.5,                    // p2_fwhm
                minIntensity + 10,      // bg_start
                minIntensity + 5        // bg_end
            ];

            // Definir límites para los parámetros, adaptados al rango de tus datos.
            // [p1_amp, p1_cen, p1_fwhm, p2_amp, p2_cen, p2_fwhm, bg_start, bg_end]
            const lowerBounds = [
                0.01,                   // p1_amp (no puede ser negativo, pero casi cero)
                minEnergy,              // p1_cen (dentro del rango de los datos)
                0.5,                    // p1_fwhm (debe ser positivo y razonable)
                0.01,                   // p2_amp
                minEnergy,              // p2_cen
                0.5,                    // p2_fwhm
                Math.min(minIntensity, 0), // bg_start (puede ser negativo si la línea base baja mucho)
                Math.min(minIntensity, 0)  // bg_end
            ]; 
            const upperBounds = [
                maxIntensity * 1.5,     // p1_amp (hasta 1.5 veces la intensidad máxima)
                maxEnergy,              // p1_cen (dentro del rango de los datos)
                4.0,                    // p1_fwhm (FWHM máximo razonable)
                maxIntensity * 1.5,     // p2_amp
                maxEnergy,              // p2_cen
                4.0,                    // p2_fwhm
                maxIntensity,           // bg_start 
                maxIntensity            // bg_end
            ]; 

            try {
                // Realizar la optimización usando Levenberg-Marquardt
                const solution = await optimization.minimize(
                    (params) => calculateSumOfSquaredResiduals(params),
                    initialParams,
                    {
                        method: 'levenberg_marquardt',
                        maxIterations: 10000, // Aumentado para mayor convergencia
                        tolerance: 1e-10,    // Tolerancia más estricta
                        lowerBounds: lowerBounds,
                        upperBounds: upperBounds
                    }
                );

                const optimalParams = solution.x;

                // Actualizar los controles deslizantes y los valores mostrados
                paramInputs.p1_amp.value = optimalParams[0].toFixed(2);
                paramValues.p1_amp.textContent = optimalParams[0].toFixed(2);
                
                paramInputs.p1_cen.value = optimalParams[1].toFixed(3);
                paramValues.p1_cen.textContent = optimalParams[1].toFixed(3);
                
                paramInputs.p1_fwhm.value = optimalParams[2].toFixed(2);
                paramValues.p1_fwhm.textContent = optimalParams[2].toFixed(2);
                
                paramInputs.p2_amp.value = optimalParams[3].toFixed(2);
                paramValues.p2_amp.textContent = optimalParams[3].toFixed(2);
                
                paramInputs.p2_cen.value = optimalParams[4].toFixed(3);
                paramValues.p2_cen.textContent = optimalParams[4].toFixed(3);
                
                paramInputs.p2_fwhm.value = optimalParams[5].toFixed(2);
                paramValues.p2_fwhm.textContent = optimalParams[5].toFixed(2);
                
                paramInputs.bg_start.value = optimalParams[6].toFixed(0);
                paramValues.bg_start.textContent = optimalParams[6].toFixed(0);
                
                paramInputs.bg_end.value = optimalParams[7].toFixed(0);
                paramValues.bg_end.textContent = optimalParams[7].toFixed(0);

                runFit(); 
                statusText.textContent = 'Ajuste automático completado.';
                showNotification('Ajuste automático exitoso!');

            } catch (error) {
                console.error('Error durante el ajuste automático:', error);
                statusText.textContent = 'Error en el ajuste automático.';
                showNotification('Error en el ajuste automático: ' + error.message, true);
            }
        }


        function runFit() {
            // Use smoothedData for fitting
            const dataForFit = smoothedData.length > 0 ? smoothedData : rawData;
            if (dataForFit.length === 0) return; 

            const xValues = dataForFit.map(d => d.x);
            const params = {};
            Object.keys(paramInputs).forEach(key => params[key] = parseFloat(paramInputs[key].value));

            const peak1_y = xValues.map(x => gaussian(x, params.p1_amp, params.p1_cen, params.p1_fwhm));
            const peak2_y = xValues.map(x => gaussian(x, params.p2_amp, params.p2_cen, params.p2_fwhm));
            const background_y = xValues.map(x => {
                const progress = (xValues[0] - x) / (xValues[0] - xValues[xValues.length - 1]);
                return params.bg_start + progress * (params.bg_end - params.bg_start);
            });

            const totalFit = xValues.map((x, i) => peak1_y[i] + peak2_y[i] + background_y[i]);
            
            // Calculate areas
            const area1 = gaussianArea(params.p1_amp, params.p1_fwhm);
            const area2 = gaussianArea(params.p2_amp, params.p2_fwhm);
            const totalAreaPeaks = area1 + area2; 
            const areaRatio = area1 / area2;

            totalFitData = xValues.map((x, i) => ({ 
                x: x, 
                y: totalFit[i], 
                peak1_y: peak1_y[i], 
                peak2_y: peak2_y[i], 
                background_y: background_y[i] 
            }));

            const numParams = 8; 
            const reducedChi2 = calculateReducedChiSquare(dataForFit, totalFit, numParams); 
            
            fitInfo.textContent = `χ²/ν: ${reducedChi2.toFixed(3)}`;
            areaInfo.textContent = `Área P1: ${area1.toFixed(2)} | Área P2: ${area2.toFixed(2)} | Relación P1/P2: ${areaRatio.toFixed(2)}`;
            displayTotalFitData(area1, area2, areaRatio, reducedChi2); 
            interpretFitResults(params.p1_cen, params.p2_cen, area1, area2, reducedChi2, params.p1_fwhm, params.p2_fwhm); // Pass FWHM
            drawPlot(); 
        }

        function displayTotalFitData(area1, area2, areaRatio, reducedChi2) {
            if (totalFitData.length === 0) {
                fitDataOutput.innerHTML = 'No hay datos de ajuste para mostrar.';
                return;
            }

            let htmlOutput = `<b>Chi-cuadrado reducido (χ²/ν): ${reducedChi2.toFixed(3)}</b><br>`;
            htmlOutput += `<b>Área Pico 1: ${area1.toFixed(2)}</b><br>`;
            htmlOutput += `<b>Área Pico 2: ${area2.toFixed(2)}</b><br>`;
            htmlOutput += `<b>Relación de Áreas (P1/P2): ${areaRatio.toFixed(2)}</b><br><br>`;
            
            htmlOutput += '<b>Energía (eV)\tAjuste Total\tPico 1\tPico 2\tLínea Base</b><br>';
            totalFitData.forEach(d => {
                htmlOutput += `${d.x.toFixed(2)}\t${d.y.toFixed(2)}\t${d.peak1_y.toFixed(2)}\t${d.peak2_y.toFixed(2)}\t${d.background_y.toFixed(2)}\n`; 
            });
            fitDataOutput.innerHTML = htmlOutput;
        }

        /**
         * Nueva función para interpretar los resultados del ajuste, sin valores ideales fijos.
         */
        function interpretFitResults(p1_cen, p2_cen, area1, area2, reducedChi2, p1_fwhm, p2_fwhm) {
            let statusMessage = '';
            let statusClass = '';

            const currentSpinOrbitSplitting = Math.abs(p2_cen - p1_cen);
            const currentAreaRatio = area1 / area2;

            let isGoodFit = true;
            let reasons = [];

            // Evaluaciones de la calidad del ajuste y consistencia del doblete
            
            // Chi-cuadrado reducido: un valor cercano a 1 es ideal, valores > 2-3 pueden indicar un mal ajuste.
            if (reducedChi2 > 2.0) { 
                isGoodFit = false;
                reasons.push(`El chi-cuadrado reducido (${reducedChi2.toFixed(3)}) es alto, lo que sugiere un ajuste no óptimo o que el modelo no se ajusta bien a los datos. Un valor ideal es cercano a 1.`);
            }

            // Desdoblamiento espín-órbita
            if (currentSpinOrbitSplitting < 1.5 || currentSpinOrbitSplitting > 3.0) { // Ampliado para permitir flexibilidad
                isGoodFit = false;
                reasons.push(`El desdoblamiento espín-órbita (${currentSpinOrbitSplitting.toFixed(2)} eV) es inusual para un doblete 3d (~2.0-2.5 eV esperado).`);
            }

            // Relación de áreas
            if (currentAreaRatio < 1.3 || currentAreaRatio > 1.7) { // Rango alrededor de 1.5 (3:2)
                isGoodFit = false;
                reasons.push(`La relación de áreas P1/P2 (${currentAreaRatio.toFixed(2)}) es inusual para un doblete 3d (se espera ~1.5).`);
            }

            // FWHM de los picos
            if (p1_fwhm > 3.0 || p2_fwhm > 3.0) { // FWHM demasiado grande
                 isGoodFit = false;
                 reasons.push(`Uno o ambos FWHM (${p1_fwhm.toFixed(2)} eV, ${p2_fwhm.toFixed(2)} eV) son grandes, lo que puede indicar múltiples especies, desorden o bajo control instrumental.`);
            }
            if (Math.abs(p1_fwhm - p2_fwhm) > 0.5) { // FWHM significativamente diferentes
                isGoodFit = false;
                reasons.push(`Los FWHM de los picos son significativamente diferentes (P1: ${p1_fwhm.toFixed(2)} eV, P2: ${p2_fwhm.toFixed(2)} eV), lo cual es inusual para un doblete 3d.`);
            }
            if (p1_fwhm < 0.8 || p2_fwhm < 0.8) { // FWHM demasiado pequeños (podría indicar overfitting o ruido excesivo)
                isGoodFit = false;
                reasons.push(`Uno o ambos FWHM (${p1_fwhm.toFixed(2)} eV, ${p2_fwhm.toFixed(2)} eV) son muy pequeños, lo que podría indicar overfitting o un ruido muy bajo.`);
            }

            if (isGoodFit && reasons.length === 0) {
                statusMessage = '✅ **Resultados del ajuste coherentes.** Los parámetros son consistentes con un doblete de espín-órbita bien ajustado. La identidad del elemento dependerá de la energía de enlace de los picos.';
                statusClass = ''; 
            } else {
                statusMessage = '⚠️ **Advertencia: Los resultados del ajuste pueden tener problemas.** Posibles razones:';
                statusMessage += '<ul>' + reasons.map(r => `<li>${r}</li>`).join('') + '</ul>';
                statusClass = 'error'; 
            }

            elementStatus.innerHTML = statusMessage;
            elementStatus.className = isGoodFit ? 'elementStatus' : 'elementStatus error';
        }


        function resetAndRefit() {
            Object.keys(defaultFitParams).forEach(key => {
                paramInputs[key].value = defaultFitParams[key];
                if (key.includes('cen')) {
                    paramValues[key].textContent = parseFloat(defaultFitParams[key]).toFixed(3);
                } else if (key.includes('fwhm')) {
                    paramValues[key].textContent = parseFloat(defaultFitParams[key]).toFixed(2);
                } else {
                    paramValues[key].textContent = defaultFitParams[key];
                }
            });
            runFit();
            showNotification("Parámetros de ajuste restablecidos.");
        }
        
        // --- Other functions ---
        function resetAll() {
            rawData = []; 
            smoothedData = []; 
            peaks = []; 
            fitTraces = []; 
            totalFitData = [];
            Plotly.purge('chart');
            fileName.textContent = '';
            dataInfo.textContent = 'Sin datos cargados';
            statusText.textContent = 'Restablecido. Cargue un archivo para comenzar.';
            fitInfo.textContent = '';
            areaInfo.textContent = ''; 
            deconvolutionControls.style.display = 'none';
            fitDataSection.style.display = 'none'; 
            fitDataOutput.innerHTML = ''; 
            elementStatus.innerHTML = ''; // Clear element status
            showNotification('Sistema restablecido');
        }
        
        function detectAndShowPeaks() {
            // Use smoothedData for peak detection
            const dataForPeakDetection = smoothedData.length > 0 ? smoothedData : rawData;

            if (dataForPeakDetection.length === 0) {
                showNotification('No hay datos para detectar picos', true);
                return;
            }
            peaks = [];
            const threshold = 0.3;
            const minPeakHeight = Math.max(...dataForPeakDetection.map(d => d.y)) * threshold;
            const minPeakDistance = 10;
            for (let i = 1; i < dataForPeakDetection.length - 1; i++) {
                if (dataForPeakDetection[i].y > minPeakHeight && dataForPeakDetection[i].y > dataForPeakDetection[i-1].y && dataForPeakDetection[i].y > dataForPeakDetection[i+1].y) {
                    if (peaks.length === 0 || (i - peaks[peaks.length - 1].index) > minPeakDistance) {
                        peaks.push({ x: parseFloat(dataForPeakDetection[i].x.toFixed(2)), y: parseFloat(dataForPeakDetection[i].y.toFixed(2)), index: i });
                    }
                }
            }
            drawPlot();
            showNotification(`${peaks.length} picos detectados`);
            statusText.textContent = `Detección de picos completada: ${peaks.length} picos`;
        }
        
        function exportChart() {
            if (rawData.length === 0) {
                showNotification('No hay gráfica para exportar', true);
                return;
            }
            Plotly.downloadImage('chart', {
                format: 'png', width: 1200, height: 800,
                filename: 'xps_analysis_interactive_fit'
            });
            showNotification('Gráfica exportada como PNG');
        }

        function exportTotalFitData() {
            if (totalFitData.length === 0) {
                showNotification('No hay datos de ajuste para exportar.', true);
                return;
            }

            const params = {};
            Object.keys(paramInputs).forEach(key => params[key] = parseFloat(paramInputs[key].value));
            const area1 = gaussianArea(params.p1_amp, params.p1_fwhm);
            const area2 = gaussianArea(params.p2_amp, params.p2_fwhm);
            const areaRatio = area1 / area2;
            const numParams = 8;
            const totalFitY = totalFitData.map(d => d.y);
            // Use smoothedData for reduced chi-square calculation for consistency
            const reducedChi2 = calculateReducedChiSquare(smoothedData, totalFitY, numParams); 

            let csvContent = "data:text/csv;charset=utf-8,";
            
            csvContent += `Reduced Chi-Square (χ²/ν),${reducedChi2.toFixed(3)}\n`;
            csvContent += `Peak 1 Area,${area1.toFixed(2)}\n`;
            csvContent += `Peak 2 Area,${area2.toFixed(2)}\n`;
            csvContent += `Area Ratio (P1/P2),${areaRatio.toFixed(2)}\n\n`;

            csvContent += "Energy (eV),Fitted Intensity,Peak 1 Intensity,Peak 2 Intensity,Background Intensity\n";
            totalFitData.forEach(row => {
                csvContent += `${row.x.toFixed(2)},${row.y.toFixed(2)},${row.peak1_y.toFixed(2)},${row.peak2_y.toFixed(2)},${row.background_y.toFixed(2)}\n`; 
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "xps_total_fit_data.csv");
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showNotification('Datos de ajuste exportados como CSV');
        }
        
        document.addEventListener('DOMContentLoaded', () => {
            statusText.textContent = 'Sistema listo. Cargue un archivo para comenzar.';
            drawPlot(); 
        });
    </script>
</body>
</html>
