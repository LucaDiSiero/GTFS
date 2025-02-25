# GTFS
Visualizador de archivos GTFS
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Visualizador de GTFS @davidantizar</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f0f4f8;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        h1 {
            text-align: center;
            color: #1a73e8;
            font-size: 2.5em;
            margin-bottom: 30px;
            text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
        }
        #status {
            text-align: center;
            font-size: 1.1em;
            margin: 20px 0;
            color: #555;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        .tabs {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
        }
        .tab-button {
            padding: 12px 25px;
            font-size: 1.1em;
            background-color: #fff;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            transition: all 0.3s ease;
        }
        .tab-button:hover {
            background-color: #1a73e8;
            color: #fff;
            transform: translateY(-2px);
        }
        .tab-button.active {
            background-color: #1a73e8;
            color: #fff;
        }
        .tab-content {
            display: none;
            background-color: #fff;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            animation: fadeIn 0.5s ease;
        }
        .tab-content.active {
            display: block;
        }
        #map {
            height: 500px;
            width: 100%;
            border-radius: 10px;
            margin-top: 20px;
        }
        .search-bar {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 20px;
        }
        input[type="text"], select {
            padding: 10px;
            font-size: 1em;
            border-radius: 5px;
            border: 1px solid #ccc;
            width: 100%;
            max-width: 250px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #e0e0e0;
        }
        th {
            background-color: #1a73e8;
            color: #fff;
            font-weight: bold;
        }
        tr:hover {
            background-color: #f5faff;
            cursor: pointer;
        }
        .trip-row {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .trip-icon {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            display: inline-block;
        }
        input[type="file"] {
            padding: 10px;
            background-color: #fff;
            border: 2px dashed #1a73e8;
            border-radius: 10px;
            display: block;
            margin: 0 auto;
            width: 300px;
            text-align: center;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            background-color: #fff;
            padding: 20px;
            border-radius: 15px;
            max-width: 800px;
            width: 90%;
            max-height: 80vh;
            overflow-y: auto;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
            animation: slideIn 0.3s ease;
        }
        .modal-close {
            float: right;
            font-size: 1.5em;
            cursor: pointer;
            color: #1a73e8;
        }
        .timeline {
            position: relative;
            margin: 20px 0;
            padding-left: 40px;
        }
        .timeline::before {
            content: '';
            position: absolute;
            left: 20px;
            top: 0;
            bottom: 0;
            width: 2px;
            background-color: #1a73e8;
        }
        .timeline-item {
            position: relative;
            margin-bottom: 20px;
        }
        .timeline-item::before {
            content: '';
            position: absolute;
            left: -22px;
            top: 50%;
            width: 12px;
            height: 12px;
            background-color: #1a73e8;
            border-radius: 50%;
            transform: translateY(-50%);
        }
        .timeline-content {
            background-color: #f5faff;
            padding: 10px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        @keyframes slideIn {
            from { transform: translateY(-20px); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.7.1/jszip.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
</head>
<body>
    <h1>Visualizador de GTFS @davidantizar</h1>
    
    <div class="container">
        <label for="gtfsZip">Subir archivo GTFS (.zip):</label>
        <input type="file" id="gtfsZip" accept=".zip">
        <div id="status"></div>

        <div class="tabs">
            <button class="tab-button active" onclick="showTab('stops')">Paradas</button>
            <button class="tab-button" onclick="showTab('routes')">Rutas por Parada</button>
        </div>

        <div id="stops" class="tab-content active">
            <h2>Mapa de Paradas</h2>
            <div id="map"></div>
        </div>

        <div id="routes" class="tab-content">
            <h2>Rutas por Parada</h2>
            <div class="search-bar">
                <input type="text" id="stopSearch" placeholder="Buscar por parada">
                <input type="text" id="tripSearch" placeholder="Buscar por viaje">
                <input type="text" id="timeSearch" placeholder="Buscar por hora (HH:MM)">
                <select id="routeTypeFilter">
                    <option value="">Filtrar por tipo de viaje</option>
                    <option value="0">Tranvía</option>
                    <option value="1">Metro</option>
                    <option value="2">Tren</option>
                    <option value="3">Autobús</option>
                    <option value="4">Ferry</option>
                    <option value="5">Teleférico</option>
                    <option value="6">Góndola</option>
                    <option value="7">Funicular</option>
                </select>
            </div>
            <div id="routesTable"></div>
        </div>

        <div id="routeModal" class="modal">
            <div class="modal-content">
                <span class="modal-close" onclick="closeModal()">×</span>
                <h2 id="modalTitle"></h2>
                <div id="routeDetails"></div>
            </div>
        </div>
    </div>

    <script>
        const requiredFiles = {
            "agency.txt": ["agency_id", "agency_name", "agency_url", "agency_timezone"],
            "stops.txt": ["stop_id", "stop_name", "stop_lat", "stop_lon"],
            "routes.txt": ["route_id", "agency_id", "route_short_name", "route_long_name", "route_type"],
            "trips.txt": ["route_id", "service_id", "trip_id"],
            "stop_times.txt": ["trip_id", "arrival_time", "departure_time", "stop_id", "stop_sequence"],
            "calendar.txt": ["service_id", "monday", "tuesday", "wednesday", "thursday", "friday", "saturday", "sunday", "start_date", "end_date"]
        };

        let gtfsData = {};
        let map;

        // Cargar archivo .zip con tolerancia a errores
        document.getElementById('gtfsZip').addEventListener('change', function(event) {
            const file = event.target.files[0];
            const statusDiv = document.getElementById('status');
            if (!file) return;

            statusDiv.textContent = "Cargando archivo .zip...";
            gtfsData = {};
            let errors = [];

            const reader = new FileReader();
            reader.onload = function(e) {
                const zip = new JSZip();
                zip.loadAsync(e.target.result).then(function(zip) {
                    const promises = [];

                    for (const fileName in requiredFiles) {
                        if (!zip.files[fileName]) {
                            errors.push(`Advertencia: Falta el archivo requerido: ${fileName}`);
                            continue;
                        }
                        promises.push(zip.files[fileName].async("string").then(function(content) {
                            const lines = content.split('\n').map(line => line.trim()).filter(line => line.length > 0);
                            if (lines.length === 0) {
                                errors.push(`Advertencia: El archivo ${fileName} está vacío`);
                                return;
                            }

                            const headers = lines[0].split(',').map(h => h.trim().replace(/^"|"$/g, '').toLowerCase());
                            const requiredHeaders = requiredFiles[fileName].map(h => h.toLowerCase());
                            for (const header of requiredHeaders) {
                                if (!headers.includes(header)) {
                                    errors.push(`Advertencia: Falta el encabezado "${header}" en ${fileName}. Encabezados encontrados: ${headers.join(', ')}`);
                                    return;
                                }
                            }

                            const data = lines.slice(1).map(row => {
                                const values = row.split(/(?<=^[^"]*(?:"[^"]*"[^"]*)*),/g).map(v => v.trim().replace(/^"|"$/g, ''));
                                const obj = {};
                                headers.forEach((header, index) => {
                                    obj[header] = values[index] || '';
                                });
                                return obj;
                            });

                            gtfsData[fileName.replace('.txt', '')] = data;
                        }).catch(err => {
                            errors.push(`Error al procesar ${fileName}: ${err.message}`);
                        }));
                    }

                    Promise.all(promises).then(function() {
                        if (errors.length > 0) {
                            statusDiv.innerHTML = "Datos cargados con advertencias:<br>" + errors.join('<br>');
                        } else {
                            statusDiv.textContent = "Datos GTFS cargados correctamente.";
                        }
                        if (gtfsData.stops) displayMap();
                        setupRoutesSearch();
                    }).catch(function(error) {
                        statusDiv.textContent = "Error crítico al procesar el archivo .zip.";
                        console.error(error);
                    });
                }).catch(function() {
                    statusDiv.textContent = "Error: No se pudo leer el archivo .zip.";
                });
            };
            reader.readAsArrayBuffer(file);
        });

        // Mostrar paradas en el mapa
        function displayMap() {
            const mapDiv = document.getElementById('map');
            mapDiv.innerHTML = '';
            if (!gtfsData.stops) return;

            map = L.map(mapDiv).setView([0, 0], 2);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap contributors'
            }).addTo(map);

            const markers = [];
            gtfsData.stops.forEach(stop => {
                const lat = parseFloat(stop.stop_lat);
                const lon = parseFloat(stop.stop_lon);
                if (!isNaN(lat) && !isNaN(lon)) {
                    const marker = L.marker([lat, lon]).addTo(map);
                    marker.bindPopup(`<b>${stop.stop_name}</b><br>ID: ${stop.stop_id}`);
                    marker.on('click', () => {
                        showTab('routes');
                        document.getElementById('stopSearch').value = stop.stop_name;
                        document.getElementById('tripSearch').value = '';
                        document.getElementById('timeSearch').value = '';
                        document.getElementById('routeTypeFilter').value = '';
                        updateRoutesTable();
                    });
                    markers.push(marker);
                }
            });

            if (markers.length > 0) {
                const group = new L.featureGroup(markers);
                map.fitBounds(group.getBounds());
            }
        }

        // Configurar búsqueda y visualización de rutas
        function setupRoutesSearch() {
            const stopSearch = document.getElementById('stopSearch');
            const tripSearch = document.getElementById('tripSearch');
            const timeSearch = document.getElementById('timeSearch');
            const routeTypeFilter = document.getElementById('routeTypeFilter');
            const routesTableDiv = document.getElementById('routesTable');

            function updateRoutesTable() {
                routesTableDiv.innerHTML = '';
                const stopText = stopSearch.value.toLowerCase();
                const tripText = tripSearch.value.toLowerCase();
                const timeText = timeSearch.value.toLowerCase();
                const routeType = routeTypeFilter.value;

                if (!gtfsData.stops || !gtfsData.stop_times || !gtfsData.trips || !gtfsData.routes) {
                    routesTableDiv.textContent = "Faltan datos necesarios para mostrar rutas.";
                    return;
                }

                let filteredStopTimes = gtfsData.stop_times;
                if (stopText) {
                    const filteredStops = gtfsData.stops.filter(stop => 
                        stop.stop_name.toLowerCase().includes(stopText) || stop.stop_id.toLowerCase().includes(stopText)
                    );
                    filteredStopTimes = filteredStopTimes.filter(st => 
                        filteredStops.some(stop => stop.stop_id === st.stop_id)
                    );
                }
                if (tripText) {
                    filteredStopTimes = filteredStopTimes.filter(st => st.trip_id.toLowerCase().includes(tripText));
                }
                if (timeText) {
                    filteredStopTimes = filteredStopTimes.filter(st => 
                        st.arrival_time.includes(timeText) || st.departure_time.includes(timeText)
                    );
                }

                const table = createTable(['Parada', 'Ruta', 'Viaje', 'Llegada', 'Salida']);
                let tripCount = 0;
                const seenTrips = new Set();

                filteredStopTimes.forEach(st => {
                    if (tripCount >= 20) return;
                    if (seenTrips.has(st.trip_id)) return;

                    const trip = gtfsData.trips.find(t => t.trip_id === st.trip_id);
                    if (!trip) return;

                    const route = gtfsData.routes.find(r => r.route_id === trip.route_id);
                    if (!route || (routeType && route.route_type !== routeType)) return;

                    const stop = gtfsData.stops.find(s => s.stop_id === st.stop_id) || { stop_name: st.stop_id };
                    const routeName = route.route_long_name || route.route_short_name;
                    const typeColor = getRouteTypeColor(route.route_type);
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td>${stop.stop_name} (${stop.stop_id})</td>
                        <td class="trip-row"><span class="trip-icon" style="background-color: ${typeColor}"></span>${routeName}</td>
                        <td>${trip.trip_id}</td>
                        <td>${st.arrival_time}</td>
                        <td>${st.departure_time}</td>
                    `;
                    row.addEventListener('click', () => showRouteDetails(trip.trip_id, routeName, typeColor));
                    table.querySelector('tbody').appendChild(row);

                    seenTrips.add(st.trip_id);
                    tripCount++;
                });

                if (tripCount === 0) {
                    routesTableDiv.textContent = "No se encontraron viajes con esos criterios.";
                } else if (tripCount >= 20) {
                    routesTableDiv.innerHTML += '<p style="color: #555;">Mostrando 20 viajes (límite alcanzado).</p>';
                }
                routesTableDiv.appendChild(table);
            }

            stopSearch.addEventListener('input', updateRoutesTable);
            tripSearch.addEventListener('input', updateRoutesTable);
            timeSearch.addEventListener('input', updateRoutesTable);
            routeTypeFilter.addEventListener('change', updateRoutesTable);
            updateRoutesTable();
        }

        // Mostrar detalles de la ruta en un modal
        function showRouteDetails(tripId, routeName, typeColor) {
            const modal = document.getElementById('routeModal');
            const modalTitle = document.getElementById('modalTitle');
            const routeDetails = document.getElementById('routeDetails');

            modalTitle.innerHTML = `<span class="trip-icon" style="background-color: ${typeColor}"></span> ${routeName} (Viaje ${tripId})`;
            routeDetails.innerHTML = '';

            const stopTimes = gtfsData.stop_times.filter(st => st.trip_id === tripId);
            if (stopTimes.length === 0) {
                routeDetails.textContent = "No hay horarios disponibles para este viaje.";
                modal.style.display = 'flex';
                return;
            }

            stopTimes.sort((a, b) => parseInt(a.stop_sequence) - parseInt(b.stop_sequence));
            const timeline = document.createElement('div');
            timeline.className = 'timeline';

            stopTimes.forEach(st => {
                const stop = gtfsData.stops.find(s => s.stop_id === st.stop_id) || { stop_name: st.stop_id };
                const item = document.createElement('div');
                item.className = 'timeline-item';
                item.innerHTML = `
                    <div class="timeline-content">
                        <b>${stop.stop_name}</b><br>
                        Llegada: ${st.arrival_time}<br>
                        Salida: ${st.departure_time}
                    </div>
                `;
                timeline.appendChild(item);
            });

            routeDetails.appendChild(timeline);
            modal.style.display = 'flex';
        }

        function closeModal() {
            document.getElementById('routeModal').style.display = 'none';
        }

        // Colores por tipo de viaje
        function getRouteTypeColor(routeType) {
            const colors = {
                '0': '#e91e63', // Tranvía: Rosa
                '1': '#2196f3', // Metro: Azul
                '2': '#4caf50', // Tren: Verde
                '3': '#ff9800', // Autobús: Naranja
                '4': '#00bcd4', // Ferry: Cyan
                '5': '#9c27b0', // Teleférico: Púrpura
                '6': '#ff5722', // Góndola: Rojo oscuro
                '7': '#795548'  // Funicular: Marrón
            };
            return colors[routeType] || '#757575'; // Gris por defecto
        }

        // Funciones auxiliares para tablas
        function createTable(headers) {
            const table = document.createElement('table');
            const thead = document.createElement('thead');
            const tbody = document.createElement('tbody');
            table.appendChild(thead);
            table.appendChild(tbody);

            const headerRow = document.createElement('tr');
            headers.forEach(header => {
                const th = document.createElement('th');
                th.textContent = header;
                headerRow.appendChild(th);
            });
            thead.appendChild(headerRow);
            return table;
        }

        // Gestión de pestañas
        function showTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.remove('active');
            });
            document.querySelectorAll('.tab-button').forEach(btn => {
                btn.classList.remove('active');
            });

            document.getElementById(tabId).classList.add('active');
            document.querySelector(`button[onclick="showTab('${tabId}')"]`).classList.add('active');
            if (tabId === 'stops' && map) map.invalidateSize();
        }
    </script>
</body>
</html>
