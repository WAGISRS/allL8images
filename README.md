How to download all Landsat8 satellite images in a time frame

# 1. Connect to Google Earth Engine
      import ee
      ee.Authenticate()
      ee.Initialize(project='your project name')

# 2. Import packages
      import folium
      import requests
      import os
      import datetime

# 3. Set study area coordinates
      aoi = ee.Geometry.Rectangle([52.45, 36.67, 52.64, 36.72])

# 4. Show area on map
      map = folium.Map(location=[aoi.centroid().getInfo()['coordinates'][1], aoi.centroid().getInfo()['coordinates'][0]], zoom_start=12)
      folium.GeoJson(aoi.getInfo(), name="AOI").add_to(map)
      map

# 5. Select tima frame
      start_date = ee.Date('2022-01-01')
      end_date = ee.Date('2022-03-31')

# 6. Import image collection
      image_collection=ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') \
            .filterBounds(aoi) \
            .filterDate(start_date, end_date)

# 7. Apply image scale factor
      def applyScaleFactors(image) :
        opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
        thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
        return image.addBands(opticalBands, None, True).addBands(thermalBands, None, True)

      image_collection = image_collection.map(applyScaleFactors);

# 8. Get image list
      image_list = image_collection.toList(image_collection.size())

# 9. Download images
      for i in range(image_list.size().getInfo()):
          image = ee.Image(image_list.get(i))
      
          # Export image date
          date = ee.Date(image.get('system:time_start')).format('YYYY-MM-dd').getInfo()

# 10. Show download links
      for i in range(image_list.size().getInfo()):
          image = ee.Image(image_list.get(i))
          date = ee.Date(image.get('system:time_start')).format('YYYY-MM-dd').getInfo()
          url = image.getDownloadURL({
              'name': f'landsat_image_{date}',
              'scale': 30,
              'crs': 'EPSG:4326',
              'region': aoi
          })
          print(f"download image {i} date {date} by link: {url}")
