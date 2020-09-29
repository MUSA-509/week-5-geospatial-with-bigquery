# Week 5 Lecture - Geospatial with BigQuery

* [Outline](#Outline)
* [Demos](#Demos)

## Outline

1. What is BigQuery?
  - Petabyte-scale "Data warehouse" that processes queries "massively in parallel"
  - Part of Google Cloud Platform
  - All public datasets can be accessed without importing them, just referencing them correctly in the query
2. Features
  - SaaS - fully managed for you
  - How does it compare with PostGIS (and other offerings)? <https://ual.sg/post/2020/07/03/a-comparison-of-spatial-functions-postgis-athena-prestodb-bigquery-vs-redshift/>
  - [Machine Learning through SQL](https://cloud.google.com/bigquery-ml/docs/introduction)
3. Why are we talking about this?
  - It may turn out that you want data bigger than is appropriate for your own PostGIS instance
  - Easier to manage than your own PostGIS
  - Handles queries and datasets in seconds that would easily break Carto
4. Access to datasets
  - [Public data](https://cloud.google.com/bigquery/public-data)
5. BigQuery GIS
  - Still under active development, not fully supported or as feature-filled as PostGIS
  - everything is a geography type!
  - all measurement functions take meters as input, or return answers in meters
6. Query formats, referencing tables
7. [GIS functions](https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions)
8. Introduction to the console
  - Query editor
  - projects list, pins
  - Pinning public datasets
  - schema/details/preview
  - query history, job history
  - geoviz, data studio
9. Demos / examples
  1. Iowa Liquor Sales
  2. Covid Open Data
  3. Geographic clustering of wildfires

## Demos

### Iowa Liquor sales

[Dataset](https://console.cloud.google.com/marketplace/product/iowa-department-of-commerce/iowa-liquor-sales)

Find the volume of alcohol consumed per population...
* within county?
* within a distance of store?

**Census tracts by volume of liquor sold per population**
```SQL
with cte as (SELECT
  c.geo_id, ST_AsText(c.tract_geom) as g,
  sum(total_volume_sold) / acs.total_pop as sum_total_volume_sold_per_pop,
  sum(total_sales) as sum_total_sales,
  count(iowa.store_number) as num_stores
FROM (
  SELECT ST_GeogFromText(store_location) as geom, store_number, sum(volume_sold_liters) as total_volume_sold, sum( sale_dollars) as total_sales
  FROM `bigquery-public-data.iowa_liquor_sales.sales`
  WHERE Extract(year from date) = 2017
  GROUP BY store_location, store_number
) as iowa
JOIN `bigquery-public-data.geo_census_tracts.census_tracts_iowa` as c
ON ST_Intersects(geom, c.tract_geom)
JOIN `bigquery-public-data.census_bureau_acs.censustract_2018_5yr` as acs
ON c.geo_id = acs.geo_id
GROUP BY 1, 2, acs.total_pop
)
SELECT *, ST_GeogFromText(g) as geom
FROM cte
ORDER BY sum_total_volume_sold_per_pop DESC
```

### Covid

```SQL
SELECT
  date,
  country_code,
  SUM(new_confirmed) as num_new_confirmed
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_code in ('DE', 'GB', 'ES')
  and new_confirmed is not null
GROUP BY date, country_code
ORDER BY date ASC
```

### Wildfire Clustering

**View the data on a map**

```SQL
SELECT ST_GeogPoint(longitude, latitude) as geom, bright_ti4, bright_ti5
FROM `bigquery-public-data.nasa_wildfire.past_week`
```

**Cluster data into regions**

```SQL
WITH Geos as (
  SELECT ST_GeogPoint(longitude, latitude) as geom, bright_ti4, bright_ti5, row_number() over () as row_id
  FROM `bigquery-public-data.nasa_wildfire.past_week`    
), clustering as (
  SELECT row_id, geom, ST_CLUSTERDBSCAN(geom, 1e4, 1) OVER () AS cluster_num FROM
  Geos ORDER BY row_id
)
SELECT
  cluster_num, ST_ConvexHull(ST_Union_Agg(geom)) as geom, count(cluster_num) as num_areas
FROM clustering
GROUP BY 1
HAVING count(cluster_num) > 20
ORDER BY 3 DESC
```
