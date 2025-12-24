// =======================================================
// AI-Based Ghost Infrastructure Detection
// Step 1: Study Area + Environment Test
// =======================================================

// Define study area (example: Pune outskirts)
// You can change coordinates later
var region = ee.Geometry.Rectangle([73.6, 18.3, 74.0, 18.7]);

// Center map
Map.centerObject(region, 10);

// Add study area layer
Map.addLayer(region, {color: 'blue'}, 'Study Area');

// Test layer to confirm editor is working
Map.addLayer(ee.Image(1), {}, 'Editor Working');
// =======================================================
// STEP 2: Sentinel-2 Built-up Verification (FIXED)
// =======================================================

// Load Sentinel-2 imagery (select required bands only)
var s2 = ee.ImageCollection('COPERNICUS/S2')
  .filterBounds(region)
  .filterDate('2024-01-01', '2024-06-01')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
  .select(['B8', 'B11'])   // IMPORTANT FIX
  .median();

// Built-up index (SWIR vs NIR)
var builtup = s2.normalizedDifference(['B11', 'B8'])
  .rename('BuiltUp');

// Visualization
Map.addLayer(builtup, {
  min: -1,
  max: 1,
  palette: ['blue', 'white', 'brown']
}, 'Built-up Index');

// =======================================================
// STEP 3: Human Activity Detection (Night-Time Lights)
// =======================================================

// Load VIIRS Night-Time Lights
var viirs = ee.ImageCollection('NOAA/VIIRS/DNB/MONTHLY_V1/VCMSLCFG')
  .filterBounds(region)
  .filterDate('2024-01-01', '2024-06-01')
  .mean()
  .select('avg_rad');

// Visualize night lights
Map.addLayer(viirs, {
  min: 0,
  max: 40,
  palette: ['black', 'yellow', 'white']
}, 'Night-Time Lights');
// =======================================================
// STEP 4: Temporal Activity Detection (Landsat Time-Series)
// =======================================================

// Load Landsat 8 imagery (long-term)
var landsat = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
  .filterBounds(region)
  .filterDate('2015-01-01', '2024-01-01')
  .select(['SR_B5', 'SR_B4']) // NIR, Red
  .map(function(img) {
    var ndvi = img.normalizedDifference(['SR_B5', 'SR_B4'])
      .rename('NDVI');
    return ndvi.copyProperties(img, ['system:time_start']);
  });

// Measure temporal variation
var temporalChange = landsat.reduce(ee.Reducer.stdDev());

// Visualize temporal activity
Map.addLayer(temporalChange, {
  min: 0,
  max: 0.2,
  palette: ['black', 'orange', 'red']
}, 'Temporal Activity');
// =======================================================
// STEP 5: Infrastructure Utilization Index (IUI)
// =======================================================

// Normalize inputs
var lightsN = viirs.unitScale(0, 40);
var temporalN = temporalChange.unitScale(0, 0.2);
var builtupN = builtup.unitScale(-1, 1);

// Compute IUI
var IUI = lightsN
  .multiply(temporalN)
  .multiply(builtupN)
  .rename('IUI');

// Visualize IUI
Map.addLayer(IUI, {
  min: 0,
  max: 0.4,
  palette: ['red', 'yellow', 'green']
}, 'Infrastructure Utilization Index');
// =======================================================
// STEP 6: AI-Based Clustering (K-Means)
// =======================================================

// Stack features
var features = ee.Image.cat([
  lightsN.rename('lights'),
  temporalN.rename('temporal'),
  builtupN.rename('builtup')
]);

// Sample pixels
var training = features.sample({
  region: region,
  scale: 500,
  numPixels: 3000
});

// Train K-Means
var clusterer = ee.Clusterer.wekaKMeans(3).train(training);

// Apply clustering
var clusters = features.cluster(clusterer);

// Visualize clusters
Map.addLayer(clusters, {
  min: 0,
  max: 2,
  palette: ['red', 'yellow', 'green']
}, 'AI Ghost Infrastructure Clusters');
