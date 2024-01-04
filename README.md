# The Delight Dispatch ☀️🗞️
The Delight Dispatch uses a news API and transformer models for sentiment analysis to deliver positive news everyday!

Demo: https://huggingface.co/spaces/DelightNews/the-delight-dispatch-demo \
Video presentation of demo: 

NOTE: Unfortunately the news API we are using will make changes on January 15th, so we have therefore created a video presentation to demonstrate our final product, but our demo on Hugging Face will have reduced content as a result of the news API change.

## Application Overview
In this project, we created a severless ML system for displaying positive news each day. Our system consists of three components:
A feature pipeline for preparing the data and inserting it into a feature group
An inference pipeline for determining the most positive article every day
A user interface for displaying the most positive article of the day as well as historical sentiment data of the retrieved news articles
To retrieve news articles daily, we used the newsdata.io API. 
In the following sections, we explain our data and the different components of our system in more detail. 

## Data
For this project, we retrieved articles written in English from different newspapers and from different countries (Great Britain, US, Australia, New Zealand, Canada, and Ireland) using the [newsdata.io API](https://newsdata.io/). 
The API returns the following attributes for each requested article: 
*'article_id', 'title', 'link', 'keywords', 'creator', 'video_url', 'description', 'content', 'pubDate', 'image_url', 'source_id', 'source_priority', 'country', 'category', 'language', 'ai_tag', 'sentiment', 'sentiment_stats'*.


## System Architecture
### Feature Pipeline
In our feature pipeline, we prepared the data to obtain reusable features which we stored in a feature group on Hopsworks. Therefore, we checked the different attributes for missing values. As we noticed during some preliminary experiments that the attributes *'keywords'*, *'creator'*, *'video_url'*, *'image_url'*, and *'source_priority'*, often contained missing values and as we did not consider these attributes as important for our goal of determining the most positive news, we decided to omit these attributes, i.e. to not include those in the feature store. Furthermore, meaningful values for the attributes *'ai_tag'*, *'sentiment'*, and *'sentiment_stats'* are only provided for premium (i.e. paying) users of the newsdata.io API. Thus, we omitted these attributes as well. Additionally to excluding attributes that were not of value for our task, we casted the attributes *'category'*, *'country'*, and *'pubDate*' to be of type *string*. 

### Inference Pipeline
In our inference pipeline, we get the data for the stored articles of the day and use a sentiment analysis transformer model to compute the sentiments of the articles. These sentiments are stored in a feature group that just contains the attributes: *'article_id'*, *'pubdate'*, *'sentiment'*. We use [this model](https://huggingface.co/distilbert-base-uncased-finetuned-sst-2-english) for sentiment analysis as it is a relatively small but still well-performing model (https://paperswithcode.com/paper/distilbert-a-distilled-version-of-bert).

We then also find the most positive article and compute the average sentiment rating of the day, which is stored along with the data for the most positive article in a feature group. This feature group stores the positive articles that are displayed in the demo and is also used to create the timeline plots for the demo of average sentiment and most positive article sentiment for the past few days. The text content of the most positive articles is summarized (before storing it in the feature group) using [this model](https://huggingface.co/sshleifer/distilbart-cnn-12-6), however since article content may exceed the token limit of the model; code posted by Moritz Laurer in the Hugging Face forums is used split the content into batches and then combine the summaries for each batch (https://discuss.huggingface.co/t/summarization-on-long-documents/920/24?page=2).

The inference pipeline also makes a call to the OpenAI API to request an image to be generated by the DALLE-3 model for the headline of the day. If the call works and an image is returned it is then stored in Hopsworks and displayed in the demo along with the article, however, the OpenAI API refuses to generate images for certain headlines and in such a case the demo will just use a default image of a logo for the app.

### User Interface
The UI for the application is a Gradio app run in a Hugging Face Space. The UI displays the most positive article of the day, an image to go with the article, and timeline plots of the average sentiment of news and the sentiment of articles shown in UI to demonstrate that the articles shown in our UI are more positive than the general news.

## Discussion
### Evaluation of our Results
Our subjective judgment is that the articles that are selected as the most positive article of the day are generally quite positive, so the system works as intended in this sense. However, sometimes the articles selected are mostly informative and sometimes specific to a local area rather than being a generally relevant and uplifting article which would be more ideal for our app.

The text content of the articles from the API also sometimes contains a lot of self-referential information about the source of the article such as links to social media accounts or information about subscriptions, but we hope that our summarization method will omit most of this information from the summary we display.

### UI Adaptions due to API changes
As mentioned above, the API used to retrieve articles daily will change the data provided for users using a *Free* plan and thus will not provide the full content of articles anymore. As we used the full article to create the summary displayed on our UI on Hugging Face, we will omit the summary from our UI. However, all other parts displayed on the UI will remain the same. 

### Possible Improvements and Extensions
#### Sentiment Rating
We only use the headline of an article to compute its sentiment, meaning that a misleading headline might cause an incorrect sentiment rating to be assigned to the article. A more robust method could be to extract some important sentences from the article and use the sentiment of these to create an overall sentiment rating for the article.

#### Image generation
The OpenAI API sometimes refuses to create images for headlines, for example, due to controversial topics or people being featured in the headline. Furthermore, the API does not support negative prompts specifying elements that should not be featured in the images, for example, we would ideally specify text or typography not to be included as the text generated often has misspellings or misformed letters. It is therefore possible that exploring some other image generation models or APIs with more features for handling the mentioned issues could provide better or more consistent images for the UI.

#### UI Extensions 
Currently, the UI only has one positive article, an image to go with it, and timeline plots for average sentiment and most positive sentiment history. However, it is possible to conceive of many extensions to the UI, such as having several articles from different categories (i.e. sports, entertainment, etc.) or adding more plots to show the sentiment of articles from different countries or newspapers, showing the total sentiment history of stored articles as a probability distribution over how likely different sentiments are, and so on.

