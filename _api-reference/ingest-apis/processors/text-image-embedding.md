---
layout: default
title: Text/image embedding
parent: Ingest processors 
grand_parent: Ingest APIs
nav_order: 270
---

# Text/image embedding

The `text_image_embedding` processor is used to generate combined vector embeddings from text and image fields for [neural search]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/). 

**PREREQUISITE**<br>
Before using the `text_image_embedding` processor, you must set up a machine learning (ML) model and provide the model ID when creating the processor.
For more information, see [ML Framework]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) and [Semantic search]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/).
{: .note}

The following is the syntax for the `text_image_embedding` processor: 

```json
{
  "text_image_embedding": {
    "model_id": "<model_id>",
    "embedding": "<vector_field>",
    "field_map": {
      "text": "<input_text_field>",
      "image": "<input_image_field>"
    }
  }
}
```
{% include copy-curl.html %}

## Parameters

The following table lists the required and optional parameters for the `text_image_embedding` processor.

| Name  | Data type | Required  | Description  |
|:---|:---|:---|:---|
`model_id` | String | Required | The ID of the model that will be used to generate the embeddings. The model must be indexed in OpenSearch before it can be used in neural search. For more information, see [ML Framework]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) and [Semantic search]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/).
`embedding` | String | Required | The name of the vector field in which to store the generated embeddings. A single embedding is generated for both `text` and `image` fields.
`field_map` | Object | Required | Contains key/value pairs that specify the fields from which to generate embeddings.
`field_map.text` | String | Optional | The name of the field from which to obtain text for generating vector embeddings. You must specify at least one `text` or `image`.
`field_map.image`  | String | Optional | The name of the field from which to obtain the image for generating vector embeddings. You must specify at least one `text` or `image`.
`description`  | String | Optional  | A brief description of the processor.  |
`tag` | String | Optional | An identifier tag for the processor. Useful for debugging to distinguish between processors of the same type. |

## Using the processor

Follow these steps to use the processor in a pipeline. You must provide a model ID when creating the processor. For more information, see [ML Framework]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/). 

**Step 1: Create a pipeline.** 

The following example request creates an ingest pipeline where the text from `image_description` and the image from `image_binary` will be converted into vector embeddings and the embeddings will be stored in `image_embedding`:

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A text/image embedding pipeline",
  "processors": [
    {
      "text_image_embedding": {
        "model_id": "bQ1J8ooBpBj3wT4HVUsb",
        "embedding": "image_embedding",
        "field_map": {
          "text": "image_description",
          "image": "image_binary"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

**Step 2 (Optional): Test the pipeline.**

It is recommended that you test your pipeline before you ingest documents.
{: .tip}

To test the pipeline, run the following query:

```json
POST _ingest/pipeline/nlp-ingest-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "image_description": "hello world",
         "image_binary": "bGlkaHQtd29rfx43..."
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### Response

The response confirms that in addition to the `image_description` and `image_binary` fields, the processor has generated vector embeddings in the `image_embedding` field:

```json
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "image_embedding": [
            -0.048237972,
            -0.07612712,
            0.3262124,
            ...
            -0.16352308
          ],
          "image_description": "hello world",
          "image_binary": "bGlkaHQtd29rfx43"
        },
        "_ingest": {
          "timestamp": "2023-10-05T15:15:19.691345393Z"
        }
      }
    }
  ]
}
```

## Next steps

- To learn more about neural search, see [Neural search]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/).
- To learn more about using models in OpenSearch, see [ML framework]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/).
- For a semantic search tutorial, see [Semantic search]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search/).