# Final Project: Retail Investor Sentiment and the S&P 500
Group Members: Val Alvern, Thiyaghessan, Pranathi Iyer, Allison Towey

# Project Scope
In recent years, the emergence of fintech applications like Robinhood have given retail investors with expanded access to the stock market. This has caused an explosion in day trading, with numerous small investors allocating increasing amounts of capital to the stock market. These investors used platforms such as reddit to share research and discuss trading strategies. One such subreddit r/wallstreetbets soon became the most popular forum, featuring millions of users. 

In early 2021, members of r/wallstreetbets observed that several hedge funds were holding large short positions against GameStop Corp, a consumer electronics and video game retailer. This meant that these financial firms were betting that the stock price would go down and these bets were exerting downward pressure on GameStop's stock price. Retail traders on r/wallstreetbets felt that this was an unfair evaluation of GameStop's business model. Several users figured out that if they drove the price of the stock up by buying shares in it, the hedge funds shorting Gamestop would be forced to pay up for their bets thus creating a feedback loop of upward price pressure. Through a combination of memes and detailed reddit discussions, millions of r/wallstreebets users simultaneously promoted and executed this strategy. The results were successful and came as a shock to institutional actors who had long dismissed the effect retail investors had on the market.

To better understand the influence ordinary investors have on the stock market currently, this research project explores the sentiment of posts within r/wallstreetbets to determine if it correlates with shifts in the S&P 500. We selected r/wallstreetbets due to its large userbase of over 12 million members and proximity to recent stock market activity as illustrated by the GameStop short squeeze. Our project scrapes the 100 most popular posts each day over a period of one year. We then host our scraped data on an API for other researchers to access it. For our analysis, we perform sentiment analysis on the text of each post before correlating it with the closing price and changes in the S&P500. Finally, we generate plots to capture this informationand hosts this plots on an interactive plotly dashboard.

# Scraping Posts From r/wallstreetbets (Thiyaghessan)
I scrape posts from reddit using the Pushift Reddit API that provides users with full functionality for parsing through reddit submissions and comments. Our timeframe for scraping spans 370 days which requires our scraper to access data from 370 unique Pushift URLs. Each API call returns data from about 100 posts in a JSON format, resulting in over 37,000 records for our scraper to parsethrough. A serial implementation is likely to take an excessive amount of time. Thus, I implement an embarassingly parallel solution using a stepfunction to split a batch of urls across 10 lambda workers. Each worker then iterates through its allocated portion of reddit posts and extracts those which possess a body of text valid for sentiment analysis. The data collected from each reddit post is its url, author, user-defined category, text of post, popularity, number of comments and number of upvotes.

The data is stored in an AWS RDS. I chose an RDS because I also want to host the contents of the dataset with a Flask API. RDS are most suited for performing multiple small and fast queries. Hence, multiple users can open connections with the database to read in the necessary rows. RDS makes this possible through its prioritization of consistency.

My scraper ends up returning about 10,000 posts. This means that over 27,000 potential posts were not scraped because they did not containt text. Their content could come in the form of videos or images. Although our earlier proposal wanted to perform an image analysis of the images used, our findings did not reveal anything particularly interesting. Thus we switched to analysing the text content instead. 

# Hosting RDS Using Flask API (Thiyaghessan)
After storing the relevant data, I create an API using Flask and host it using an AWS EBS Cluster. Detailed instructions for accessing the API are provided in the attached notebooks. The aim of making the dataset accessible via API is to provide other researchers with a  means of working with relevant subsections of our data. For example, our API allows a user to index rows by date, author or category. This provides an easy format for other researchers to navigate r/wallstreetbets posts that contain text in their body. Our scraper has already filtered out non-text posts and deleted posts so the results from our dataset are much cleaner than the results from the Reddit API.

# Sentiment Analysis (Val)

# Statistical Analysis (Pranathi)

# Creation of Interactive Plots (Allison)
