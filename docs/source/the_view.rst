.. _customization:

The Slick Report View and form
===============================

What is SlickReportView?
-----------------------

SlickReportView is a CBV that inherits form ``FromView`` and expose the report generator needed attributes.
it also

* Auto generate the search form
* return the results as a json response if ajax request
* Works on GET and POST

How the search form is generated ?
-----------------------------------
Behind the scene, Sample report calls ``slick_reporting.form_factory.report_form_factory``
a helper method which generates a form containing start date and end date, as well as all foreign keys on the report_model.

Override the Form
------------------

The Form purposes are

1. Provide the start_date and end_date
2. provide a ``get_filters`` method which return a tuple (Q_filers , kwargs filter) to be used in filtering.
   q_filter: can be none or a series of Django's Q queries
   kwargs_filter: None or a dict of filters

Following those 2 recommendation, your awesome custom form will work as you'd expect.


Charting
---------

Charts settings is a list of objects which each object represent a chart configurations.

* type: what kind of chart it is: Possible options are bar, pie, line and others subject of the underlying charting engine.
  Hats off to : `Charts.js <https://www.chartjs.org/>`_.
* engine_name: String, default to ``SLICK_REPORTING_DEFAULT_CHARTS_ENGINE``. Passed to front end in order to use the appropriate chart engine.
  By default supports `highcharts` & `chartsjs`.
* data_source: Field name containing the numbers we want to plot.
* title_source: Field name containing labels of the data_source
* title: the Chart title. Defaults to the `report_title`.
* plot_total if True the chart will plot the total of the columns. Useful with time series and crosstab reports.

On front end, for each chart needed we pass the whole response to the relevant chart helper function and it handles the rest.

The ajax response structure
---------------------------

Understanding how the response is structured is imperative in order to customize how the report is displayed on the front end.

Let's have a look

.. code-block:: python


    # Ajax response or `report_results` template context variable.
    response = {
        # the report slug, defaults to the class name all lower
        "report_slug": "",

        # a list of objects representing the actual results of the report
        "data": [
            {"name": "Product 0", "quantity__sum": "1774", "value__sum": "8758", "field_n" : "value_n"},
            # .....
        ],

        # A list explaining the columns/keys in the data results.
        # ie: len(response.columns) == len(response.data[i].keys())
        # Contains needed information about the verbose name , if summable , hints about the data type.
        "columns": [
            {"name": "name",
             "computation_field": "",
             "verbose_name": "Name",
             "visible": True,
             "type": "CharField",
             "is_summable": False
             },
            {"name": "quantity__sum",
             "computation_field": "",
             "verbose_name": "Quantities Sold",
             "visible": True,
             "type": "number",
             "is_summable": True},
            {"name": "value__sum",
             "computation_field": "",
             "verbose_name": "Value $",
             "visible": True,
             "type": "number",
             "is_summable": True}
        ],

        # Contains information about the report as whole if it's time series or a a crosstab
        # And what's the actual and verbose names of the time series or crosstab specific columns.
        "metadata": {"time_series_pattern": "",
                     "time_series_column_names": [],
                     "time_series_column_verbose_names": [],
                     "crosstab_model": '',
                     "crosstab_column_names": [],
                     "crosstab_column_verbose_names": []
                     },


        # a mirror of the set charts_settings on the SlickReportView
        # SlickReportView populates the id if missing and fill the `engine_name' if not set
        "chart_settings": [
            {"type": "pie",
            'engine_name': 'highcharts',
             "data_source": ["quantity__sum"],
             "title_source": ["name"],
             "title": "Pie Chart (Quantities)",
             "id": "pie-0"},

            {"type": "bar",
            "engine_name": "chartsjs",
            "data_source": ["value__sum"],
            "title_source": ["name"],
            "title": "Column Chart (Values)",
             "id": "bar-1"}
        ]
    }


