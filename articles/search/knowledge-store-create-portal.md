---
title: "Quickstart: Create a knowledge store in the Azure portal"
titleSuffix: Azure Cognitive Search
description: Use the Import data wizard to create a knowledge store used for persisting enriched content. Connect to a knowledge store for analysis from other apps, or send enriched content to downstream processes.

author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 09/02/2021
---

# Quickstart: Create a knowledge store in the Azure portal

Knowledge store is a feature of Azure Cognitive Search that sends skillset output from an [AI enrichment pipeline](cognitive-search-concept-intro.md) to Azure Storage for subsequent analysis or downstream processing.

An enrichment pipeline accepts unstructured text and image content, applies AI-powered processing by Cognitive Services, and outputs new structures and information that didn't previously exist. One of the physical data structures created by a pipeline is a [knowledge store](knowledge-store-concept-intro.md), which you can access through tools that analyze and explore content in Azure Storage such as [Storage Explorer](knowledge-store-view-storage-explorer.md) or [Power BI](knowledge-store-connect-power-bi.md).

In this quickstart, you'll set up your data and then run the **Import data** wizard to create an enrichment pipeline that also generates a knowledge store. The knowledge store will contain original text content pulled from the source, plus AI-generated content that includes a sentiment score, key phrase extraction, and text translation of non-English customer comments.

> [!NOTE]
> This quickstart is the fastest route to a finished knowledge store in Azure Storage. For more detailed coverage, see [Create a knowledge store in REST](knowledge-store-create-rest.md) instead.

## Prerequisites

This quickstart uses the following services:

+ An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/).

+ An Azure Cognitive Search service. [Create a service](search-create-service-portal.md) or [find an existing service](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) in your account. You can use a free service for this quickstart. 

+ An Azure Storage account with [Blob Storage](../storage/blobs/index.yml).

This quickstart also uses [Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) for the AI. Because the workload is so small, Cognitive Services is tapped behind the scenes for free processing for up to 20 transactions. This means that you can complete this exercise without having to create an additional Cognitive Services resource.

## Set up your data

In the following steps, set up a blob container in Azure Storage to store heterogeneous content files.

1. [Download HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?sp=r&st=2019-11-04T01:23:53Z&se=2025-11-04T16:00:00Z&spr=https&sv=2019-02-02&sr=b&sig=siQgWOnI%2FDamhwOgxmj11qwBqqtKMaztQKFNqWx00AY%3D). This data is hotel review data saved in a CSV file (originates from Kaggle.com) and contains 19 pieces of customer feedback about a single hotel. 

1. [Create an Azure storage account](../storage/common/storage-account-create.md?tabs=azure-portal) or [find an existing account](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/). You'll use Azure Storage for both the raw content to be imported, and the knowledge store that is the end result.

   Choose the **StorageV2 (general purpose V2)** account type.

1. In the Azure Storage resource, use **Storage Explorer** to create a blob container named **hotel-reviews**.

1. Select **Upload** at the top of the page to load the **HotelReviews-Free.csv** file you downloaded from the previous step.

   :::image type="content" source="media/knowledge-store-create-portal/blob-container-storage-explorer.png" alt-text="Screenshot of Storage Explorer with uploaded file and left nav pane" border="true":::

1. You are almost done with this resource, but before you leave these pages, select **Access Keys** on the left navigation pane to get a connection string so that you can retrieve this data using the indexer.

1. In **Access Keys**, select **Show Keys** at the top of the page to unhide the connection strings, and then copy the connection string for either key1 or key2.

   A connection string has the following format: `DefaultEndpointsProtocol=https;AccountName=<YOUR-ACCOUNT-NAME>;AccountKey=<YOUR-ACCOUNT-KEY>;EndpointSuffix=core.windows.net`

You are now ready to move on the **Import data** wizard.

## Start the wizard

1. Sign in to the [Azure portal](https://portal.azure.com/) with your Azure account.

1. [Find your search service](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/) and on the Overview page, click **Import data** on the command bar to create a knowledge store in four steps.

   :::image type="content" source="media/search-import-data-portal/import-data-cmd.png" alt-text="Screenshot of the Import data command" border="true":::

### Step 1: Create a data source

Because the data is multiple rows in one CSV file, set the *parsing mode* to get one search document for each row.

1. In **Connect to your data**, choose **Azure Blob storage**, selecting the account and container you created. 

1. For the **Name**, enter `hotel-reviews-ds`.

1. For **Parsing mode**, select **Delimited text**, and then select the **First Line Contains Header** checkbox. Make sure the **Delimiter character** is a comma (,).

1. In **Connection String**, paste in the connection string you copied from Azure Storage.

1. In **Containers**, enter the name of the blob container holding the data (`hotel-reviews`).

    Your page should look similar to the following screenshot.

   :::image type="content" source="media/knowledge-store-create-portal/hotel-reviews-ds.png" alt-text="Screenshot of data source definition" border="true":::

1. Continue to the next page.

### Step 2: Add skills

In this wizard step, add skills for AI enrichment. The source data consists of customer reviews in English and French. Skills that are relevant for this data set include key phrase extraction, sentiment detection, and text translation. In a later step, these enrichments will be "projected" into a knowledge store as Azure tables.

1. Expand **Attach Cognitive Services**. **Free (Limited enrichments)** is selected by default. You can use this resource because the number of records in HotelReviews-Free.csv is 19 and this free resource allows up to 20 transactions a day.

1. Expand **Add enrichments**.

1. For **Skillset name**, enter `hotel-reviews-ss`.

1. For **Source data field**, select **reviews_text**.

1. For **Enrichment granularity level**, select **Pages (5000 characters chunks)**.

1. For **Text Cognitive Skills**, select the following skills:

    + **Extract key phrases**
    + **Translate text**
    + **Language detection**
    + **Detect sentiment**

   Your page should look like the following screenshot:

   :::image type="content" source="media/knowledge-store-create-portal/hotel-reviews-ss.png" alt-text="Screenshot of the skillset definition" border="true":::

1. Scroll down and expand **Save enrichments to knowledge store**.

1. Select these **Azure table projections**:
    + **Documents**
    + **Pages**
    + **Key phrases**

1. Enter the **Storage account Connection String** that you saved in a previous step.

   :::image type="content" source="media/knowledge-store-create-portal/hotel-reviews-ks.png" alt-text="Screenshot of the knowledge store definition" border="true":::

1. Continue to the next page.

### Step 3: Configure the index

In this wizard step, configure an index for optional full-text search queries. The wizard will sample your data source to infer fields and data types. You only need to select the attributes for your desired behavior. For example, the **Retrievable** attribute will allow the search service to return a field value while the **Searchable** will enable full text search on the field.

1. For **Index name**, enter `hotel-reviews-idx`.

1. For attributes, accept the default selections: **Retrievable** and **Searchable** for the new fields that the pipeline is creating.

    Your index should look similar to the following image. Because the list is long, not all fields are visible in the image.

   :::image type="content" source="media/knowledge-store-create-portal/hotel-reviews-idx.png" alt-text="Screenshot of the index definition" border="true":::

1. Continue to the next page.

### Step 4: Configure and run the indexer

In this wizard step, configure an indexer that will pull together the data source, skillset, and the index you defined in the previous wizard steps.

1. For **Name**, enter `hotel-reviews-idxr`.

1. For **Schedule**, keep the default **Once**.

1. Click **Submit** to run the indexer. Data extraction, indexing, application of cognitive skills all happen in this step.

## Check indexer status

In the **Overview** page, open the **Indexers** tab in the middle of the page, and then select **hotels-reviews-ixr**. Within a minute or two, status should progress from "In progress" to "Success" with zero errors and warnings.

## Check tables in Azure Storage

In the Azure portal, switch to your Azure Storage account and use **Storage Explorer** to view the new tables. You should see six tables.

Each table is generated with the IDs necessary for cross-linking the tables in queries. When you open a table, scroll past these fields to view the content fields added by the pipeline.

   :::image type="content" source="media/knowledge-store-create-rest/knowledge-store-tables.png" alt-text="Screenshot of the knowledge store tables in Storage Explorer" border="true":::

In this walkthrough, the knowledge store is composed of a various tables showing different ways of shaping and structuring a table. The first three rely on a Shaper skill to provide the columns and data values.

| Table | Description |
|-------|-------------|
| hotelReviews1Document | Contains fields carried forward from the CSV, such as reviews_date and reviews_text. |
| hotelReviews2Pages | Contains enriched fields created by the skillset, such as sentiment score and translated text. |
| hotelReviews3KeyPhrases | Contains a long list of just the key phrases. |
| hotelReviews4InlineProjectionDocument | Alternative to the first table, using inline shaping instead of the Shaper skill to shape data for the projection. |
| hotelReviews5InlineProjectionPages | Alternative to the second table, using inline shaping. |
| hotelreviews6InlineProjectionKeyPhrases | Alternative to the third table, using inline shaping. |

## Clean up

When you're working in your own subscription, it's a good idea at the end of a project to identify whether you still need the resources you created. Resources left running can cost you money. You can delete resources individually or delete the resource group to delete the entire set of resources.

You can find and manage resources in the portal, using the **All resources** or **Resource groups** link in the left-navigation pane.

If you are using a free service, remember that you are limited to three indexes, indexers, and data sources. You can delete individual items in the portal to stay under the limit.

> [!TIP]
> If you want to repeat this exercise or try a different AI enrichment walkthrough, delete the **hotel-reviews-idxr** indexer and the related objects to recreate them. Deleting the indexer resets the free daily transaction counter to zero.

## Next steps

Now that you've enriched your data by using Cognitive Services and projected the results to a knowledge store, you can use Storage Explorer or other apps to explore your enriched data set.

To learn how to explore this knowledge store by using Storage Explorer, see this walkthrough:

> [!div class="nextstepaction"]
> [View with Storage Explorer](knowledge-store-view-storage-explorer.md)
