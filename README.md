# Geospatial with BigQuery

* [Homework](#Homework)
* Lecture
  * [Slide deck](https://docs.google.com/presentation/d/1H3kZH8zwgeU5o7EQV0Ctcq3LkyaEMSSvPZ6gfTkWd3E/edit?usp=sharing)
  * [Lecture notes](Lecture.md)
* Lab - posted Thursday

## Homework

- Continue working on Homework 2 (due on Oct 6th at 11:59pm ET)
- [Intro to BigQuery GIS tutorial](https://cloud.google.com/bigquery/docs/gis-getting-started)
- [BigQuery covid blog post](https://medium.com/google-cloud/analyzing-covid-19-with-bigquery-13701a3a785) - I encourage you to follow along and recreate all the work, up to the forecasting part.
- Practice some BQ queries
  - If you were born in the United States, try to find yourself in the [natality dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=samples&t=natality&page=table). If you weren't born in the US, try to find a friend if they don't mind sharing some of their personal details.
  - What was the average weather in the world on your birthday? Use the [`gsod` dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=samples&t=gsod&page=table) of weather readings around the world. E.g., what was the total amount of precipitation measured?

## Lab Outline

1. Query public datsets
2. Visualize using the GeoViz tool
3. Create a new dataset from a dataset in Google Storage
4. Mix public and private datasets in a query
