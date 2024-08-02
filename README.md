# Metadata-Driven Databricks Project

This project use databricks and metadata-driven approach. 
The ETL transformations are defined in a JSON file, and the Databricks notebook reads the metadata to know source, target and transform rule.

Some param in project set by workflow and job
    such as: "auto" and "set_date"


## Structure in S3

Metadata
```
cmg-metadata-demo/
├── schema/
│   └── schema.csv
├── raw-to-silver/
│   └── classify_config.json\
│       ...
└── silver-to-gold/
    └── customer-scd2.json
        ...
```


Raw data
```
metadata_etl_project/
├── oracle/
│   ├── integration-date=2024-08-01/
│   │           ..
│   └── integration-date=2024-08-02/
│       └── Product.csv
│       └── Customer.csv
│       └── SalesOrder.csv
│       ...
├── mysql/
└── ...
```


## Step and Run

1. **Create catalog cmg_gold_zone_demo,cmg_silver_zone_demo, meta_data_set**
2. **Create table data_linage with code**
3. **Set and run job in Workflow step by step**
    cmg-mount-data --> raw-to-silver-demo --> silver-to-gold-demo


## Metadata Configuration

The transformations rule are defined in a JSON file (`cmg-metadata-demo/raw-to-silver/classify_config.json`). 
The JSON structure includes source and target details along with the transformations to apply:

```json
{
    "id": "classify_data_by_schema",
    "source": {
        "type": "csv",
        "path": "cmg-raw-data-demo",
        "delimiter": ",",
        "header": true,
        "encoding": "utf-8"
    },
    "target": {
        "type": "delta",
        "catalog_path": "cmg_silver_zone_demo",
        "mode": "append"
    },
    "transform": {
        "classify": {
            "type": "csv",
            "path": "cmg-metadata-demo/schema/schema.csv",
            "delimiter": ",",
            "header": true,
            "encoding": "utf-8"
        },
        "filter": {}
    }
}
```


JSON file (`cmg-metadata-demo/raw-to-silver/customer-scd2.json`):
```json
{
    "id": "silver-to-gold-customer-scd2",
    "source": {
        "catalog_path": "cmg_silver_zone_demo",
        "schema_name": "sales",
        "table_name": "customer",
        "type": "delta"
    },
    "target": {
        "catalog_path": "cmg_gold_zone_demo",
        "schema_name": "sales",
        "table_name": "dwh_customer",
        "type": "delta"
    },
    "transform": {
        "type": "scd2",
        "key": "CustomerID"
    }
}
```
