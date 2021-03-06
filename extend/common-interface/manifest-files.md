---
title: Manifest Files Specification
permalink: /extend/common-interface/manifest-files/
---

* TOC
{:toc}

A manifest file contains additional information about tables and files injected to the
[`/data/in` folders](/extend/common-interface/).
It also provides a way to specify options for tables and files transferred back to Storage from `/data/out`
folders. Manifest files have a `.manifest` suffix to the original file.

All files in `/data/in` have the manifest file mandatory and injected. For files generated by your code
in `/data/out`, the manifest file **is optional**. Also keep in mind that all manifests have a lower priority
than an input and output mapping.

## Format

The format of the manifest file is always *JSON*. The manifest
file always has the `.manifest` extension.

## Examples

### Tables

#### `/data/in/tables` Manifests
An input table manifests stores metadata about a downloaded table. For example, a table
with ID `in.c-docker-demo.data` will be downloaded into
`/in/tables/in.c-docker-demo.data.csv' (unless stated otherwise in the
[input mapping](/extend/common-interface/config-file/)] and a manifest file
'/in/tables/in.c-docker-demo.data.csv.manifest' will be created with the following
contents:

{% highlight json %}
{
    "id": "in.c-docker-demo.data",
    "uri": "https://connection.keboola.com//v2/storage/tables/in.c-docker-demo.data",
    "name": "data",
    "primary_key": [],
    "indexed_columns": [],
    "created": "2015-01-25T01:35:14+0100",
    "last_change_date": "2015-01-25T01:35:14+0100",
    "last_import_date": "2015-01-25T01:35:14+0100",
    "rows_count": 2,
    "data_size_bytes": 32768,
    "is_alias": false,
    "columns": [
        "id",
        "name",
        "text"
    ],
    "attributes": []
}
{% endhighlight %}

The `name` node refers to the name of the component configuration.
Note that `data_size_bytes` and `rows_count` values are estimated by the database server and they may be
significantly off (especially right after the table is created). The `attributes` node contains
additional [table attributes](https://help.keboola.com/storage/). If used, it has the following structure:

{% highlight json %}
{
    ...
    "atrributes": [
        {
            "name": "attributeName",
            "value": "attributeValue",
            "protected": false
        }
    ]
}
{% endhighlight %}

#### `/data/out/tables` Manifests

An output table manifest sets options for transferring a table to Storage. The following examples list available
manifest fields, **all of them are optional**. `destination` field overrides table name generated
from file name, it can (and commonly is) overridden by end-user configuration.

{% highlight json %}
{
    "destination": "out.c-main.Leads",
    "columns": ["column1", "column2", "column3"],
    "incremental": true,
    "primary_key": ["column1", "column2"],
    "delimiter": "\t",
    "enclosure": "\""
}
{% endhighlight %}

Additionally, the following options can be specified:

{% highlight json %}
{
    ...
    "delete_where_column": "column name",
    "delete_where_values": ["value1", "value2"],
    "delete_where_operator": "eq"
}
{% endhighlight %}

Which will cause to be the specified rows deleted from the source table before the new
table is imported, see [Example](/extend/common-interface/config-file/#output-mapping---delete-rows).

### Files

#### `/data/in/files` Manifests

An input file manifest stores metadata about a downloaded file.

{% highlight json %}
{
    "id": 75807657,
    "created": "2015-01-14T00:47:00+0100",
    "is_public": false,
    "is_sliced": false,
    "is_encrypted": true,
    "name": "fooBar.jpg",
    "size_bytes": 563416,
    "tags": [
        "tag1",
        "tag2"
    ],
    "max_age_days": 180,
    "creator_token": {
        "id": 3800,
        "description": "ondrej.hlavacek@keboola.com"
    }
}
{% endhighlight %}

#### `/data/out/files` Manifests

An output file manifest sets options for transferring a file to Storage. The following example lists available
manifest fields, all of them are optional.

{% highlight json %}
{
    "is_public": true,
    "is_permanent": true,
    "is_encrypted": true,
    "notify": false,
    "tags": [
        "image",
        "pie-chart"
    ]
}
{% endhighlight %}

These parameters can be used (taken from [Storage API File Import](http://docs.keboola.apiary.io/#files)):
If `is_permnanent` is false, then the file will be automatically deleted after 180 days. When `notify` is
true, the members of the project will be notified that a file has been uploaded to the project.

