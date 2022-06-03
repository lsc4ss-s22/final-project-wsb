# Final Project: Retail Investor Sentiment and the S&P 500
Group Members: Val Alvern, Thiyaghessan, Pranathi Iyer, Allison Towey

# Project Scope
In recent years, the emergence of fintech applications like Robinhood have given retail investors with expanded access to the stock market. This has caused an explosion in day trading, with numerous small investors allocating increasing amounts of capital to the stock market. These investors used platforms such as reddit to share research and discuss trading strategies. One such subreddit r/wallstreetbets soon became the most popular forum, featuring millions of users. 

In early 2021, members of r/wallstreetbets observed that several hedge funds were holding large short positions against GameStop Corp, a consumer electronics and video game retailer. This meant that these financial firms were betting that the stock price would go down and these bets were exerting downward pressure on GameStop's stock price. Retail traders on r/wallstreetbets felt that this was an unfair evaluation of GameStop's business model. Several users figured out that if they drove the price of the stock up by buying shares in it, the hedge funds shorting Gamestop would be forced to pay up for their bets thus creating a feedback loop of upward price pressure. Through a combination of memes and detailed reddit discussions, millions of r/wallstreebets users simultaneously promoted and executed this strategy. The results were successful and came as a shock to institutional actors who had long dismissed the effect retail investors had on the market.

To better understand the influence ordinary investors have on the stock market currently, this research project explores the sentiment of posts within r/wallstreetbets to determine if it correlates with shifts in the S&P 500. We selected r/wallstreetbets due to its large userbase of over 12 million members and proximity to recent stock market activity as illustrated by the GameStop short squeeze. Our project scrapes the 100 most popular posts each day over a period of one year. We then host our scraped data on an API for other researchers to access it. For our analysis, we perform sentiment analysis on the text of each post before correlating it with the closing price and changes in the S&P500. Additionally, we also implement an LDA topic modelling algorithm to identify possible themes that guide investor sentiment. Finally, we generate plots to capture this informationand hosts this plots on an interactive plotly dashboard.

Answering this question has important implications for current scholarship on behavioural economics and behavioural finance. It provides insight into how the manifest emotions of day traders can exert an influence on markets. Additionally, it also provides a measure of the magnitude of this influence. Beyond its findings, this project offers scholars with a repository of reddit posts from r/wallstreetbets and a parallelized framework to use when analyzing the data. For example, the scraper can be used to retrieve posts across a longer time frame for an extended analysis.

# Scraping Posts From r/wallstreetbets
I scrape posts from reddit using the Pushift Reddit API that provides users with full functionality for parsing through reddit submissions and comments. Our timeframe for scraping spans 370 days which requires our scraper to access data from 370 unique Pushift URLs. Each API call returns data from about 100 posts in a JSON format, resulting in over 37,000 records for our scraper to parsethrough. A serial implementation is likely to take an excessive amount of time. Thus, I implement an embarassingly parallel solution using a stepfunction to split a batch of urls across 10 lambda workers. Each worker then iterates through its allocated portion of reddit posts and extracts those which possess a body of text valid for sentiment analysis. The data collected from each reddit post is its url, author, user-defined category, text of post, popularity, number of comments and number of upvotes.

The data is stored in an AWS RDS. I chose an RDS because I also want to host the contents of the dataset with a Flask API. RDS are most suited for performing multiple small and fast queries. Hence, multiple users can open connections with the database to read in the necessary rows. RDS makes this possible through its prioritization of consistency. Aside from RDS, I also output the results into a csv file which I then upload to an S3 bucket for the rest of the team to access.

My scraper ends up returning about 10,000 posts. This means that over 27,000 potential posts were not scraped because they did not containt text. Their content could come in the form of videos or images. Although our earlier proposal wanted to perform an image analysis of the images used, our findings did not reveal anything particularly interesting. Thus we switched to analysing the text content instead. Although the scrapers only returned 10,000 valid posts our code should not have an issue scraping larger volumes of data. All that is required is for the user to change the date range in the jupyter notebook.

Jupter Notebook for scraping: [LSC_Final_Project_Scraper.ipynb](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/LSC_Final_Project_Scraper.ipynb) <br>
Lambda Function for scraping: [wsb_lambda.py](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/wsb_lambda.py) <br>
Step Function for scraping: [wsb_sfn.py](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/wsb_sfn.py) <br>

# Hosting RDS Using Flask API
After storing the relevant data, I create an API using Flask and host it using an AWS EBS Cluster. Detailed instructions for accessing the API are provided in the attached notebooks. The aim of making the dataset accessible via API is to provide other researchers with a  means of working with relevant subsections of our data. For example, our API allows a user to index rows by date, author or category. This provides an easy format for other researchers to navigate r/wallstreetbets posts that contain text in their body. Our scraper has already filtered out non-text posts and deleted posts so the results from our dataset are much cleaner than the results from the Reddit API. Instructions on how to use the API can be found on its homepage using the index.html template. Although I have terminated the EBS cluster the code used is available in this repository.

Python Script for API: [LSC_Final_Project_API.ipynb](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/LSC_Final_Project_API.py) <br>
Zip File for AWS EBS: [LSC_Final_Project_API.zip](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/LSC_Final_Project_API.zip) <br>

# Topic Modelling
We perform LDA topic modelling on the data to identify the core themes that guide the evolution of the subreddit and potentially reveal drivers of retail investor sentiment. To implement this algorithm we use Pyspark NLP in an AWS EMR cluster that performs topic modelling on a series of unigrams and trigrams. We also use POS tagging to filter out uninformative sequences of words before fitting the topic model on the combined set of unigrams and trigrams. 

The results of the topic models largely reference various market activites relevant to the buying and selling of stock. The most interesting topic is topic 6 with words like "elon", "buy", "uranium" and "shib". "Shib" references the Shiba Inu, the dog that is the face of Doge Coin. This coupled with the mentioning of Elon Musk and the nonsensical use of uranium (It is not accessible as a commodity to be traded) suggests that there is a possible strong element of humour amongst users of the subreddit. This is inline with qualitative explorations of r/wallstreetbets where users often create posts as a form of satire.

Additionally, there are also several clear favourite stock picks amongst users such as "gme" (Gamestop) and blackberry. The former reveals that the early events of 2021 continue to exert an influence on discourse within the subreddit. The latter suggests that users on the forumn do promote specific companies and this could be a part of efforts to inflate the share price of certain companies. The success of these efforts can be evaluated in further research.

The fact that the rest of the topics center around market activities also suggests that retail traders are as capable of performing dedicated research as their institutional counterparts. A common criticism of retail investors by financial institutions is their relative unsophistication and irrationality. However, our results suggest that retail traders do use the approrpriate vocabulary when discussing the markets and are focused on valid metrics. This does call into question assumptions of incompetence institutional actors and scholars have often held when studying retail investors. The results are shown below: <br>
![Final_Project_LDA.png](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/Final_Project_LDA.png)

Pyspark Notebook for Topic Modelling: [LSC_Final_Project_LDA.ipynb](https://github.com/lsc4ss-s22/final-project-wsb/blob/main/LSC_Final_Project_LDA.py) <br>

# Sentiment Analysis (Val)
I implement a sentiment analysis model from John Snow labs to facilitate further statistical analysis of the individual reddit posts. The model in question was a pre-trained BERT model, first trained on a wikipedia corpus before then being fine-tuned to identify sentiment in financial statements. Specifically, the encodings are sentence embeddings from the Bert model. After importing the scraped data from Thiya's S3 bucket, I isolate the 'text' column in the csv before preprocessing the text, removing URLs, long strings of numbers and special characters. In the process of preprocessing, one data point was removed. Thus, we had to remove the corresponding datapoint in the original dataframe to be able to easily match each result to its date. The preprocessed text is then put through the sentiment analysis model. The results from the model is then concatenated with the corresponding date to facilitate the time-series statistical analysis by Pranathi. The final dataframe is then saved onto an S3 bucket for future use. 

# Statistical Analysis (Pranathi)

# Creation of Interactive Plots (Allison)
