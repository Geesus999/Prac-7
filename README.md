
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Web Map with Relocatable Pink Pin and POIs</title>
    <!-- Include the ArcGIS Maps SDK for JavaScript CSS and JavaScript -->
    <link rel="stylesheet" href="https://js.arcgis.com/4.26/esri/themes/light/main.css">
    <script src="https://js.arcgis.com/4.26/"></script>
    <style>
        #viewDiv {
            width: 100vw;
            height: 100vh;
        }
        .custom-legend {
            position: absolute;
            bottom: 10px;
            right: 10px;
            background: white;
            padding: 10px;
            border-radius: 4px;
            box-shadow: 0 0 6px rgba(0, 0, 0, 0.3);
            font-family: Arial, sans-serif;
        }
        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 5px;
        }
        .legend-symbol {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            margin-right: 5px;
        }
        .blue-marker { background-color: blue; border: 1px solid white; }
        .green-marker { background-color: green; border: 1px solid black; }
        .purple-marker { background-color: purple; border: 1px solid white; }
        .red-marker { background-color: red; border: 1px solid white; }
        .cyan-marker { background-color: cyan; border: 1px solid white; }
        .teleport-buttons {
            position: absolute;
            top: 10px;
            right: 10px;
            display: flex;
            gap: 10px;
        }
        .teleport-button {
            background-color: white;
            padding: 8px 15px;
            border: 1px solid #0079c1;
            color: #0079c1;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.2);
        }
        .teleport-button:hover { background-color: #0079c1; color: white; }
    </style>
</head>
<body>
    <div id="viewDiv"></div>
    
    <!-- Custom Legend -->
    <div class="custom-legend">
        <div class="legend-item"><div class="legend-symbol blue-marker"></div><span>Adelaide</span></div>
        <div class="legend-item"><div class="legend-symbol green-marker"></div><span>Second Location</span></div>
        <div class="legend-item"><div class="legend-symbol purple-marker"></div><span>Glenelg Beach</span></div>
        <div class="legend-item"><div class="legend-symbol red-marker"></div><span>Cafe</span></div>
        <div class="legend-item"><div class="legend-symbol cyan-marker"></div><span>Waterfall</span></div>
    </div>

    <!-- Teleport Buttons -->
    <div class="teleport-buttons">
        <div class="teleport-button" id="teleportMadrid">Go to Madrid</div>
        <div class="teleport-button" id="teleportAdelaide">Go to Adelaide</div>
        <div class="teleport-button" id="relocatePin">Relocate Pin</div>
        <div class="teleport-button" id="findPin">Find Pin</div>
    </div>

    <script>
        require([
            "esri/Map",
            "esri/views/SceneView",
            "esri/layers/GraphicsLayer",
            "esri/Graphic",
            "esri/widgets/BasemapToggle",
            "esri/widgets/ScaleBar",
            "esri/geometry/Point"
        ], function(Map, SceneView, GraphicsLayer, Graphic, BasemapToggle, ScaleBar, Point) {

            // Create a map with a default basemap
            const map = new Map({ basemap: "topo-vector" });

            // Set up the initial view as a SceneView (3D)
            const view = new SceneView({
                container: "viewDiv",
                map: map,
                center: [0, 0], // Start at the center of the Earth
                zoom: 2
            });

            // Add a BasemapToggle widget
            const basemapToggle = new BasemapToggle({
                view: view,
                nextBasemap: "satellite"
            });
            view.ui.add(basemapToggle, "top-right");

            // Add a ScaleBar widget
            const scaleBar = new ScaleBar({
                view: view,
                unit: "metric"
            });
            view.ui.add(scaleBar, "bottom-left");

            // Create a graphics layer for points and pin
            const graphicsLayer = new GraphicsLayer();
            map.add(graphicsLayer);

            let pinkPin; // Variable to hold the pink pin graphic

            // Function to generate a random location on Earth
            function getRandomLocation() {
                const randomLongitude = (Math.random() * 360) - 180; // Longitude between -180 and 180
                const randomLatitude = (Math.random() * 180) - 90;   // Latitude between -90 and 90
                return [randomLongitude, randomLatitude];
            }

            // Function to relocate the pink pin
            function relocatePin() {
                const [longitude, latitude] = getRandomLocation();

                // Remove the existing pink pin if it exists
                graphicsLayer.remove(pinkPin);

                // Create a new pink pin graphic
                pinkPin = new Graphic({
                    geometry: new Point({
                        longitude: longitude,
                        latitude: latitude
                    }),
                    symbol: {
                        type: "simple-marker",
                        color: "pink",
                        size: "14px",
                        outline: { color: "white", width: 1 }
                    }
                });

                // Add the pink pin to the graphics layer
                graphicsLayer.add(pinkPin);
            }

            // Event listener for the "Relocate Pin" button
            document.getElementById("relocatePin").addEventListener("click", relocatePin);

            // Event listener for the "Find Pin" button
            document.getElementById("findPin").addEventListener("click", function() {
                if (pinkPin) {
                    view.goTo({
                        target: pinkPin.geometry,
                        zoom: 5 // Zoom level to focus on the pink pin
                    });
                } else {
                    alert("Please press 'Relocate Pin' first to place the pink pin.");
                }
            });

            // Points for Adelaide, Second Location, and Glenelg Beach
            const adelaidePoint = new Graphic({
                geometry: new Point({ longitude: 138.6, latitude: -34.92 }),
                symbol: { type: "simple-marker", color: "blue", outline: { color: "white", width: 1 } },
                popupTemplate: {
                    title: "Adelaide",
                    content: `<p>This is a test pop-up with an image of Adelaide</p><img src="https://upload.wikimedia.org/wikipedia/commons/c/c3/Adelaide_-_SA_%2825091207887%29.jpg" width="200px">`
                }
            });

            const secondLocationPoint = new Graphic({
                geometry: new Point({ longitude: 138.58, latitude: -34.94 }),
                symbol: { type: "simple-marker", color: "green", outline: { color: "black", width: 1 } },
                popupTemplate: { title: "Second Location", content: "This is the second point marker" }
            });

            const glenelgPoint = new Graphic({
                geometry: new Point({ longitude: 138.507, latitude: -34.978 }),
                symbol: { type: "simple-marker", color: "purple", outline: { color: "white", width: 1 } },
                popupTemplate: {
                    title: "Glenelg Beach",
                    content: `<p>Enjoy the beautiful sands of Glenelg Beach!</p><img src="https://jooinn.com/images/sea-shell-21.jpg" width="200px">`
                }
            });

            graphicsLayer.addMany([adelaidePoint, secondLocationPoint, glenelgPoint]);

            // Button to teleport to Madrid and add POIs
            document.getElementById("teleportMadrid").addEventListener("click", function() {
                view.goTo({ center: [-3.7038, 40.4168], zoom: 13 }).then(addMadridLocations);
            });

            // Button to teleport back to Adelaide
            document.getElementById("teleportAdelaide").addEventListener("click", function() {
                view.goTo({ center: [138.6, -34.92], zoom: 12 });
            });

            // Function to add cafes and waterfalls in Madrid
            function addMadridLocations() {
                const cafes = [
                    [-3.703, 40.417], [-3.707, 40.415], [-3.704, 40.419],
                    [-3.709, 40.416], [-3.705, 40.414]
                ].map(coords => new Graphic({
                    geometry: new Point({ longitude: coords[0], latitude: coords[1] }),
                    symbol: { type: "simple-marker", color: "red", outline: { color: "white", width: 1 } },
                    popupTemplate: { title: "Cafe", content: "A nice place to enjoy coffee in Madrid." }
                }));

                const waterfalls = [
                    [-3.706, 40.420], [-3.702, 40.418], [-3.701, 40.415]
                ].map(coords => new Graphic({
                    geometry: new Point({ longitude: coords[0], latitude: coords[1] }),
                    symbol: { type: "simple-marker", color: "cyan", outline: { color: "white", width: 1 } },
                    popupTemplate: { title: "Waterfall", content: "A scenic waterfall location in Madrid." }
                }));

                graphicsLayer.addMany([...cafes, ...waterfalls]);
            }
        });
    </script>
</body>
</html>

