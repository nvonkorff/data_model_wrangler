# Data model wrangler

## Overview
Data model wrangler is a Splunk app that helps to display information about Splunk data models and the data sources mapped to them.

This is useful in environments that rely on data being correctly mapped to the Common Information Model (CIM) data models in Splunk. This is especially important for environments using Splunk Enterprise Security, which relies heavily on the Common Information Modeli data models. Lower quality CIM mapping yields lower quality security insights.

This app aims to provide the following information:
 * Data model-level view
   * Mapping quality - Percentage coverage of recommended fields (as per Splunk CIM documentation) in data mapped to each data model
   * Data quality - Percentage data quality of recommended fields (utilises CIM_Validator)
 * Field-level view
   * Per "data model, index, sourcetype" view of recommended fields vs actual mapped fields in the data (used to derive mapping quality score)
   * Per "data model, index, sourcetype, field" view of field coverage in the data, along with event count and distinct value count for each field. e.g. Authentication datamodel, index=test, sourcetype=test, field=src has 25% coverage across the 542,837 events.

## How does it work?
There are three saved searches. Each of these is scheduled to run on a periodic basis to populate a summary index.

 * data_model_wrangler_dm_index_sourcetype_field - For each data model, get a list of indexes and sourcetypes mapped, then for each index and sourcetype, get a list of data model fields available in the data along with counts for each field 
 * data_model_wrangler_field_quality - For entries in the summary index, get the field data quality by checking 
 * data_model_wrangler_mapping_quality - For entries in the summary index, get the coverage of recommended fields for each datamode, index
, sourcetype

## How do I use the app?

 * Ensure you have the [Common Information Model](https://splunkbase.splunk.com/app/1621/) app installed and up-to-date
 * Install the [CIM Validator app](https://splunkbase.splunk.com/app/2968/), as Data model wrangler relies on a custom search command from the CIM Validator app. There are also drill-downs from panels in the Data model wrangler to the CIM Validator.
 * Create an index for the data that will be generated. Default index name is: data_model_wrangler
   * If you want to use a custom index name, please update the 'data_model_wrangler_index' search macro in the app with the custom index name
 * Enable the below saved searches
   * data_model_wrangler_dm_index_sourcetype_field (default schedule is twice per day, once at 06:00 and again at 14:00. Time range: -4h@h to -3h@h)
   * data_model_wrangler_field_quality (default schedule is once per day at 01:00)  
   * data_model_wrangler_mapping_quality (default schedule is once per day at 01:00)

Over time this will build up a view of mapping and field data quality over time, allowing you to see the current state, find issues, and when issues are resolved, you can measure the results.

Whenever a CIM mapping requirement comes up, this dashboard provides a quick and easy way to find existing mapped data, or to verify that newly added data has been CIM mapped correctly.

## FAQs

 * I can't see an index or field that I know is there and has been CIM mapped
   * Verify that the index has been added to the CIM_<data_model>_indexes search macro in the CIM app






