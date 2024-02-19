# Exploring Azure AI Search index

Study case: You work for Fourth Coffee, a national coffee chain. You’re asked to help build a knowledge mining solution that makes it easy to search for insights about customer experiences. You decide to build an __Azure AI Search index__ using data extracted from customer reviews. Here's the step by step: 

- Create Azure resources
- Extract data from a data source
- Enrich data with AI skills
- Use Azure’s indexer in the Azure portal
- Query your search index
- Review results saved to a Knowledge Store

The solution requires the following resources in Azure:

- An Azure AI Search resource, which will manage indexing and querying.
- An Azure AI services resource, which provides AI services for skills that your search solution can use to enrich the data in the data source with AI-generated insights.

### 1) Let's start creating an Azure AI Search resource at [Azure Portal](https://portal.azure.com/), with the following settings:

- Subscription: Your Azure subscription.
- Resource group: Select or create a resource group with a unique name.
- Service name: A unique name.
- Location: Choose any available region.
- Pricing tier: Basic

### 2) Creating an Azure AI Services resource
We need to provision an __Azure AI Services__ resource in the same location as __Azure AI Search__ resource. The search solution will use this resource to enrich the data in the datastore with AI-generated insights.
Returning to the home page of the Azure portal, create an Azure AI Services resource. Configure it with the following settings:

- Subscription: Your Azure subscription.
- Resource group: The same resource group as Azure AI Search resource.
- Region: The same location as Azure AI Search resource.
- Name: A unique name.
- Pricing tier: Standard S0
- Select checking box: I acknowledge that I have read and understood all the terms below

### 3) Creating a storage account
Return to the home page of the Azure portal, and select the +Create a resource button.
Search for storage account, and create a Storage account resource with the following settings:

- Subscription: Your Azure subscription.
- Resource group: The same resource group as Azure AI Search and Azure AI services resources.
- Storage account name: A unique name.
- Location: Choose any available location.
- Performance: Standard
- Redundancy: Locally redundant storage (LRS)

In the Azure Storage account created, in the left-hand menu pane, select __Configuration (under Settings)__ and change the setting for "Allow Blob anonymous access" to __Enabled__ and then select Save.

### 4) Uploading Documents to Azure Storage
In the left-hand menu pane, select Containers > +Container. A pane on right-hand side opens. Enter the following settings, and click Create:

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/storage-blob-1.png)

- Name: coffee-reviews
- Public access level: Container (anonymous read access for containers and blobs)
- Advanced: no changes
- Download the zipped coffee reviews from https://aka.ms/mslearn-coffee-reviews, and extract the files to the reviews folder. In the Azure portal, select your coffee-reviews container. In the container, select Upload.
- In the Upload blob pane, select Select a file. In the Explorer window, select all the files in the reviews folder, select Open, and then select __Upload__.

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/storage-blob-3.png)

- After the upload is complete, close the Upload blob pane. Documents are now in your coffee-reviews storage container.

### 5) Index the documents
After the documents are in storage, use Azure AI Search to extract insights from the documents. The Azure portal provides an Import data wizard. With this wizard, you can automatically create an index and indexer for supported data sources. Use the wizard to create an index, and import the search documents from storage into the Azure AI Search index.

In the Azure portal, browse to your Azure AI Search resource. On the Overview page, select __Import data__.

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/azure-search-wizard-1.png)
  
On the Connect to your data page, in the Data Source list, select Azure Blob Storage. Complete the data store details with the following values:

- Data Source: Azure Blob Storage
- Data source name: coffee-customer-data
- Data to extract: Content and metadata
- Parsing mode: Default
- Connection string: *Select Choose an existing connection. Select your storage account, select the coffee-reviews container, and then click __Select__.
- Managed identity authentication: None
- Container name: this setting is auto-populated after you choose an existing connection.
- Blob folder: Leave this blank.
- Description: Reviews for Fourth Coffee shops.
- Select __Next: Add cognitive skills (Optional)__.

In the Attach Cognitive Services section, select your Azure AI services resource. In the Add enrichments section:

- Change the Skillset name to coffee-skillset.
- Select the checkbox __Enable OCR and merge all text into merged_content field__
- Ensure that the Source data field is set to __merged_content__.
- Change the Enrichment granularity level to __Pages (5000 character chunks)__.
- __Don’t__ select Enable incremental enrichment
- Select the following enriched fields:

| Cognitive Skill | Field name |  
|---|---|
| Extract location names | locations |
| Extract key phrases | keyphrases |
| Detect sentiment | sentiment |
| Generate tags from images | imageTags |
| Generate captions from images | imageCaption |
   
Under __Save enrichments to a knowledge store__, select:

Image projections\
Documents\
Pages\
Key phrases\
Entities\
Image details\
Image references

- Select Azure blob projections: __Document__. A setting for Container name with the knowledge-store container auto-populated displays. Don’t change the container name.
- Select __Next: Customize target index__. Change the Index name to __coffee-index__.
- Ensure that the Key is set to __metadata_storage_path__. Leave Suggester name blank and Search mode autopopulated.
- Review the index fields’ default settings. Select __filterable__ for all the fields that are already selected by default.

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/6a-azure-cognitive-search-customize-index.png)


- Select __Next: Create an indexer__.
- Change the Indexer name to __coffee-indexer__.
- Leave the Schedule set to __Once__.
- Expand the Advanced options. Ensure that the __Base-64 Encode Keys__ option is selected, as encoding keys can make the index more efficient.
- Select Submit to create the data source, skillset, index, and indexer. The indexer is run automatically and runs the indexing pipeline, which:  
1 Extracts the document metadata fields and content from the data source.\
2 Runs the skillset of cognitive skills to generate more enriched fields.\
3 Maps the extracted fields to the index.

Return to your Azure AI Search resource page. On the left pane, __under Search Management, select Indexers__. Select the newly created __coffee-indexer__. Wait a minute, and select &orarr; Refresh until the Status indicates success.

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/6a-search-indexer-success.png)

### 6)Querying the index
Now we can use the Search explorer to write and test queries. Search explorer is a tool built into the Azure portal that gives you an easy way to validate the quality of your search index. You can use Search explorer to write queries and review results in JSON.

- In Search service’s Overview page, select __Search explorer__ at the top of the screen.

![](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/media/create-cognitive-search-solution/5-exercise-screenshot-7.png)

- Below the index selected, change the view to __JSON view__.

We can use the search to return all the documents in the search index, including a count of all the documents in the @odata.count field. The search index should return a JSON document containing search results:

```
{
    "search": "*",
    "count": true
}
```

Let’s filter by location. The query searches all the documents in the index and filters for reviews with a Chicago location. You should see 3 in the @odata.count field:

```
{
 "search": "locations:'Chicago'",
 "count": true
}
```

Now let’s filter by sentiment. The query searches all the documents in the index and filters for reviews with a negative sentiment. You should see 1 in the @odata.count field:

```
{
 "search": "sentiment:'negative'",
 "count": true
}
```

