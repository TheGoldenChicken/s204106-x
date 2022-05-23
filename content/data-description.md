---
title: Data description
prev: "/"
next: network-analysis
---

# Downloading the data for out analysis

We choose to query from the following subreddits, to have a variraty in the biasses of redditors and to hopefully reduce the amount of _meme_ posts and comments.

* r\news
* r\television
* r\worldnews
* r\USnews
* r\qualitynews
* r\offbeat
* r\OutOfTheLoop
* r\Oscars
* r\boxoffice
* r\willsmith
* r\entertainment

Next we wanted to download every submission related to the Oscars slap. This was done using the PushshiftAPI from psaw.

We queried for all submissions with one of the follwing words in the title:
* slap
* smith
* chris
* rock
* oscar
* oscars

and between the 2022-03-28 00:00:00 (the start of the oscars ceramony) and 2022-04-17 23:59:59 (the day we downloaded the dataset).


Among the quried information was the subreddit, authorname, created time, a unique submission id and the title of the submission. This yielded 3300 submission for our submission dataset.

From here we proceded to download every comment from every submission in our submission dataset. Amongst the features downloaded were the subreddit, author name, comment body, created time, unique comment id, the id for the submission and the id for the parent comment (in case the comment is a reply to another comment). The query yielded 186 603 comments 


## Combining the two datasets

To analyse our data as a single dataset, the two datasets were concatenated in such a way, that the submissions are counted as a comment, where the comments replying to the submissions are read as child comments. 

To make this concatenation, the titles of a submission is counted as the body of a text.

This way we created 3300 trees, with submissions as root nodes, comments as children nodes, comments with no replies as leaf nodes and edges as pointers to the perent comment of the node.


# Cleaning and filtering the dataset

Firstly the body was cleaned to only contain alphanummeric values and spaces.

Secondly all comments with '[deleted]' bodies were removed as these add nothing to the descussion.

Secondly every comment pointing to a parent comment not contained in our dataset, had the parent comment set to the submission the comment is a part of.

## Filtering out bots

Any person having spent significant time on reddit, will know that bots such as u/AutoModerater helps moderate various larger subreddits. Purposes other than moderation also excists for bots. Comments made by these bots have no use in this analysis and we therefore wish to filter them out. 

Our first method was simply to count the maximum number of equal comments made by a single bot and if this number was greater than $$4 + 0.05\cdot n_{comments}$$ we would count it as one. This method came short when bots included things the link to the post, mentioned the author or the like. 

Our second idea was to find an average similarity messure of the comments made by a single author. This was done with the mean jaro distance between all combinations of two comments made by a single author. The jaro distance is optimal since it is normalized between 0 and 1, where 1 means excact match. All authors with an average jaro score of above 65% and 10 or more comments are considered bots.

![](/images/bot_plot.png)
This image shows the decision boundary for the linear bot test as well at the mean jaro similarity score. u/AutoModerator has been highlighted as it scored high in both tests and is one of the most known bots on subreddit. The Jaccard similarity was also calculated between the two tests with a resault of 6.1%.

Since the two methods seemed to differ a lot we chose to filter out data by authors choosen by either method. Only comments made by the bots were removed, where any submissions where kept. Also each comment with a bot comment as the parent, had their parent set to the parent of the bot comment to keep the overall data structure.

This lead to removing 1536 suspected bot comments, equvalent to 0.81% of the total data.


## Filtering out non topic datapoints

Since we had made a broad search for our query, we had lots of data regarding the Oscars as a whole, post involving guys named Oscar and slaps not related to the Oscars was in our dataset. Therefore, we used the reddit RegEx library to categorize which of the following five subjects our datapoints belonged to.

1. Will Smith
1. Jada Pinkett
1. Chris Rock
1. slap
1. Oscars

A datapoint was considered to belong to a certain category, if the body contained any of the words in the catagory. Will Smith was the exception, since will is a widely used stopword. This is a premitive method, but we found it effective.

Now came the obvious problem, comments can be about a subject without mentioning the subject implicitly. E.g. "He needs to learn how to take a joke", "Notice how he laughed at the joke, untill he saw her reaktion" and many more.

To solve this problem, we came up with the method **trickle down subjects**. If a comment mentions a certain subject, the whole subtree emerging from that node, will be categorized to also include that subject. This is not perfect, as subjects can change through a comment thred, but this is a compromize we chose to work with.

The datastructure, algorithm and everything else created to aplly the trickle down subjects method, can be found in the notebook.


## Filtering the data

Since the subject of each comment is now known, a quick filter could be implement to filter out non *relavant* comments.

The filter we choose to use, was the comment had to at least contain Will Smith, Jada Pinkett, Chris Rock or slap and Oscars.

Of the original 189 903 datapoints, only 21.2% would have been kept if we aplied the filter before the use of the trickle down subjects method, were 61.3% were deemed relavant after we aplied the trickle down subjects method.

This mean our cleaned dataset consists of 115 316 datapoints.


# Data analysis

![](/images/hourly_engagement.png)


<img src="/images/subject_barplot.png" width="450" />

<img src="/images/author_comment_hist.png" width="450" />
