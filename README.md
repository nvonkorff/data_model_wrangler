# Data model wrangler

## Overview
Data model wrangler is a Splunk app that helps to display information about Splunk data models and the data sources mapped to them.

This is useful in environments that rely on data being correctly mapped to the Common Information Model (CIM) data models in Splunk. This is especially important for environments using Splunk Enterprise Security, which relies heavily on the CIM data models. Lower quality CIM mapping yields lower quality security insights.

This app aims to provide the following information:

* Data model-level view
    * Mapping quality - Percentage of recommended fields (as per Splunk CIM documentation) present in the data model for data sources mapped to each data model
    * Field data quality - Percentage coverage of fields present in the underlying data (utilises CIM_Validator)
* Field-level view
    * Per "data model, index, sourcetype" view of recommended fields vs actual mapped fields in the data (used to derive mapping quality score)
    * Per "data model, index, sourcetype, field" view of field coverage in the data, along with event count and distinct value count for each field (used to derive data quality score). e.g. Authentication datamodel, index=test, sourcetype=test, field=src has 25% coverage across the 542,837 events.

## How do I use the app?

* Ensure you have the [Common Information Model](https://splunkbase.splunk.com/app/1621/) app installed and up-to-date
* Install the [CIM Validator app](https://splunkbase.splunk.com/app/2968/), as Data model wrangler relies on a custom search command from the CIM Validator app. There are also drill-downs from panels in the Data model wrangler to the CIM Validator.
* Create an index for the data that will be generated. Default index name is: *data_model_wrangler*
    * If you want to use a custom index name, please update the 'data_model_wrangler_index' search macro in the app with the custom index name
* Enable the below saved searches
    * data_model_wrangler_dm_index_sourcetype_field (default schedule is twice per day, once at 06:00 and again at 14:00. Time range: -4h@h to -3h@h)
    * data_model_wrangler_field_quality (default schedule is once per day at 01:00)  
    * data_model_wrangler_mapping_quality (default schedule is once per day at 01:00)

Over time this will build up a view of mapping and field data quality over time, allowing you to see the current state, find issues and measure the results.

Whenever a CIM mapping requirement comes up, this dashboard provides a quick and easy way to find existing mapped data, or to verify that newly added data has been CIM mapped correctly.

## How does it work?

### Saved searches
There are three saved searches. Each of these is scheduled to run on a periodic basis to populate a summary index.

* data_model_wrangler_dm_index_sourcetype_field
    * For each data model, get a list of indexes and sourcetypes mapped, then for each index and sourcetype, get a list of data model fields available in the data along with counts for each field 
* data_model_wrangler_field_quality
    * For entries in the summary index, get the field data quality by checking the coverage percentage of each field in the data, along with the distinct count of values for the field (this data comes from the CIM Validator app)
* data_model_wrangler_mapping_quality
    * For entries in the summary index, get the coverage of recommended fields for each data model, index, sourcetype

### Dashboards
The app provides the following dashboards:
* Data model wrangler
  * Provides information about data models, the data sources mapped to them. The dashboard flows top to bottom from least specific (data mdodel level) to most specific (data source field level)
* Data model review
  * Provides details from the data_model_wrangler_health_review.csv to aid with reviewing and fixing low quality data sources
* Documentation

### Lookups
The app uses two lookup files. These are:
* datamodel_recommended_fields.csv
  * Contains a list of data models and the recommended (as per the [Splunk CIM documentation](https://docs.splunk.com/Documentation/CIM/latest/User/Overview)) fields for each data model
  * Coverage of recommened fields is used to determine the mapping and 
  * Add any custom data models and recommended fields to this list to enable the app to calculate coverage scores for custom data models
* data_model_wrangler_health_review.csv
* A place to track, review and comment on data sources that are reporting low quality
* This lookup is used by the "Data model review" dashboard

### Search macros
The app uses a number of search macros to store key pieces of information and to formulate search queries. These two search macros can be customised to suit your environment:
* `data_model_wrangler_index`
  * Index to which summary data will be sent
  * Default: *data_model_wrangler*
* `datamodel_wrangler_data_model_list`
Comma separated list of data models to monitor
  * Default: *Authentication, Change, Change_Analysis, DLP, Databases, Email, Endpoint, Intrusion_Detection, Malware, Network_Resolution, Network_Sessions, Network_Traffic, Web*
* `data_model_wrangler_health_review_lookup`
  * The name of the lookup containing review information
  * Default: *data_model_wrangler_health_review.csv*

## Workflow
Typical workflow is as follows:
* Identify low mapping or field data quality data models ("Data model wrangler" dashboard)
  * Use the "Data model" drop down to select the data model to show the one you want to target
  * This shows you your Mapping, Field data and Overal quality scores for the selected data model
* Identify low mapping quality data sources within the data model ("Data model wrangler" dashboard)
  * Scroll down to the "Indexes, sourcetypes and fields mapped to data models - Splunk DM recommended field coverage" panel
  * Check for data sources that have low quality scores. You can filter by the following fields to find lowest scores: rec_field_coverage_pct, percent_data_coverage, overall_quality_pct
  * You can select a specific index or sourcetype in the filters at the top of the Data model wrangler dashboard to more specifically target certain data sources
  * For sources that do not have data model fields mapped, check your field aliases, field extractions, calculated fields for that data source to ensure these are defined correctly
* Identify low data quality data sources within the data model ("Data model wrangler" dashboard)
  * Scroll down to the "Indexes, sourcetypes and fields mapped to data models" panel
  * Select the "Only recommended fields" radio button in input of this panel to hide non-recommended fields
  * Check for fields that have low to zero coverage in the data source
  * Check your field extractions are correct, whether this source is getting incorrectly tagged, or whether it makes more sense to instead remove this data from the data model by untagging it.
* Review and track changes ("Data model review" dashboard)
  * The "Data model review" dashboard allows you to track reviews of data sources
  * There is a link to a lookup (data_model_wrangler_health_review.csv) on this dashboard that you can use to track/review progress (the link relies on the "Lookup Editor" app being installed - otherwise go to Settings>Lookups)
  * For each, data model, index, sourcetype reviewed, add a row to this .csv with the details. For reviewed items:
  * 'reviewed' column, set the value: true
  * 'status' column, set the value to one either:
    * Open
    * In Development
    * Done
  * 'comment' column - put a comment that explains the issue or a link to a JIRA task for instance

# FAQs

 * I can't see an index or field that I know is there and has been CIM mapped
    * Verify that the index has been added to the CIM_<data_model>_indexes search macro in the CIM app

## Caution

Depending on the size of your environment, some of the saved searches may run for a long time, and may be resource intensive.

## Acknowledgements
The original idea for this app was not mine, but I was chosen to build it. Thanks to the talented team of security professionals for coming up with and providing feedback on the app, and thanks to the organisation where this was developed for approving the app to be released to the public. You know who you are!

## Contribute
If you would like to contribute, please do so at the following GitHub repository: [GitHub: Data model wrangler](https://github.com/nvonkorff/data_model_wrangler) 

## Support

Support is provided on a best-effort basis in my free time.

If you find this app helpful, please feel free to tip. I accept Dogecoin. My wallet address is: DC39d1d8LgXx2U2iV8r8d3kU7ceLuSEw6u

