# 🌊 **Sentinel-1 Flood Hazard and Water Mapping** 🌊

Welcome to the **Sentinel-1 Flood Hazard and Permanent Water Mapping** project! This tool uses **Sentinel-1 SAR** (Synthetic Aperture Radar) data to assess **flood hazards** and detect **permanent water bodies** in a given region. The project analyzes seasonal changes in water coverage, helping to identify flood-prone areas and map permanent water over the years 2016 to 2023.

With this tool, you can visualize flood hazards, calculate flood areas, and track permanent water bodies in your area of interest. Results are available for **GeoTIFF** exports for spatial analysis and **CSV** for tabular flood area statistics.

---

## 🧑‍🔬 **Key Features of This Project**

- **Flood Hazard Mapping**: Identify flood-prone areas by analyzing wet and dry seasons.
- **Permanent Water Mapping**: Detect and map permanent water bodies over multiple years.
- **Flood Area Calculation**: Calculate the area of flooding for each year.
- **Data Export**: Export results as **GeoTIFF** for spatial data and **CSV** for flood area statistics.

---

## 🚀 **Getting Started**

### 1. **Sign up for Google Earth Engine (GEE)**

To run the script, you’ll need a **Google Earth Engine (GEE)** account. If you don’t have one, you can sign up [here](https://signup.earthengine.google.com/).

### 2. **Run the Script**

Once you have access to GEE, open the [Google Earth Engine Code Editor](https://code.earthengine.google.com/), paste the script, and click **Run**. The script processes **Sentinel-1 SAR data** for the years 2016-2023, providing flood hazard maps, permanent water detection, and flood area statistics for the region of interest.

---

## 🔧 **How This Works**

### 1. **Load Sentinel-1 Data**

We load the **Sentinel-1 GRD (Ground Range Detected)** dataset and filter it by the area of interest. The script selects the **VV polarization** band, which is most suitable for water detection:

```javascript
var s1 = ee.ImageCollection("COPERNICUS/S1_GRD");
```

### 2. **Define Region of Interest (ROI)**

The region of interest (ROI) is defined by a bounding polygon (`bounds1`). You can adjust this polygon to focus on a different area if needed.

```javascript
var bounds1 = ee.Geometry.Polygon(
    [[[-76.74813761234307, 41.040156813292256],
      [-76.74813761234307, 38.7992085672086],
      [-72.44149698734307, 38.7992085672086],
      [-72.44149698734307, 41.040156813292256]]], null, false);
```

### 3. **Seasonal Analysis: Wet and Dry**

The analysis focuses on two key seasons:

- **Wet season**: From December 1st to December 31st.
- **Dry season**: From August 1st to August 31st.

These dates are used to calculate the water mask for each season.

```javascript
var props = [
  { name: 'wet', start: '-12-01', end: '-12-31', palette: 'navy' },
  { name: 'dry', start: '-08-01', end: '-08-31', palette: 'lightskyblue' }
];
```

### 4. **Flood Detection**

Flood-prone areas are detected by comparing the wet and dry season water masks. Flooding occurs where water is present in the wet season but not in the dry season:

```javascript
var flood = waterWet.and(waterDry.eq(0)).rename('flood').toByte();
```

### 5. **Water Mapping**

The script also calculates **permanent water** areas by combining the wet and dry season water masks. These areas represent bodies of water that persist throughout the year.

```javascript
var waterPermanent = images.select('water').reduce(ee.Reducer.allNonZero()).and(floodHazard.mask().eq(0)).rename('water');
```

### 6. **Flood Hazard Index**

The **Flood Hazard Index** is calculated by summing the occurrences of flooding over the years and normalizing by the number of years. This index helps identify areas with frequent flooding.

```javascript
var floodHazard = images.select('flood').sum().divide(years.length);
Map.addLayer(floodHazard, { min: 0, max: 1, palette: ['white', 'pink', 'red'] }, 'Flood hazard');
```

### 7. **Flood Area Calculation**

The **flood area** is calculated for each year by multiplying pixel area by the flood mask and summing up the area (in hectares):

```javascript
var floodArea = ee.Number(ee.Image.pixelArea().multiply(1e-4).updateMask(flood).reduceRegion({
  scale: 100,
  geometry: bounds1,
  reducer: ee.Reducer.sum(),
  maxPixels: 1e13
}).get('area'));
```

### 8. **Data Export**

The following data can be exported:

- **Flood Hazard Index** as a **GeoTIFF** for spatial analysis.
- **Permanent Water** map as a **GeoTIFF**.
- **Flood Area Data** as a **CSV** file.

```javascript
Export.image.toDrive({
  image: floodHazard,
  description: 'Flood_Hazard_Index',
  folder: 'GEE_Exports',
  fileNamePrefix: 'flood_hazard_index',
  region: bounds1,
  scale: 30,
  crs: 'EPSG:4326',
  maxPixels: 1e13
});

Export.table.toDrive({
  collection: floodAreaTable,
  description: 'Flood_Area_Data',
  folder: 'GEE_Exports',
  fileNamePrefix: 'flood_area_data',
  fileFormat: 'CSV'
});
```

---

## 📊 **Visualizations and Outputs**

### 1. **Flood Hazard Map**

This map visualizes areas prone to flooding. The **Flood Hazard Index** is calculated based on how often flooding occurs:

```javascript
Map.addLayer(floodHazard, { min: 0, max: 1, palette: ['white', 'pink', 'red'] }, 'Flood hazard');
```

### 2. **Permanent Water Map**

This map shows areas with **permanent water presence**, which can include lakes, rivers, or reservoirs:

```javascript
Map.addLayer(waterPermanent.selfMask(), { palette: 'blue' }, 'Permanent water');
```

### 3. **Flood Area Chart**

A **chart** is generated to display the total **flood area (in hectares)** for each year from 2016 to 2023:

```javascript
var chart = ui.Chart.feature.byFeature(images, 'year', ['flood_area'])
  .setChartType('ColumnChart')
  .setOptions({
    title: 'Flood Area (Ha) 2016 - 2023',
    vAxis: { title: 'Flood area (Ha)' },
    hAxis: { title: 'Year' },
    series: { 0: { color: 'lightskyblue' } }
  });
print(chart);
```

---
![ee-chart (2)](https://github.com/user-attachments/assets/c67e4304-9e3b-474d-8a4a-5cf46c09602c)

![Snapshot_۲۵-۰۴-۲۸_۰۵-۳۷-۱۸](https://github.com/user-attachments/assets/f186b1d0-2904-4550-8b2c-0b13032a0f27)


## 📈 **Future Enhancements**

- **Improved Seasonal Analysis**: Customize seasonal definitions to suit different climatic regions.
- **Machine Learning**: Use machine learning models to predict flood occurrences based on **Sentinel-1** data.
- **Data Fusion**: Combine **Sentinel-1** SAR data with **Sentinel-2** optical imagery for enhanced flood detection.

---

