# Beginner's Crash Course to Elastic Stack Series
## Part-3-Running-full-text-queries-and-combined-queries-with-Elasticsearch-and-Kibana

Welcome to the Beginner's Crash Course to Elastic Stack!

This repo contains all resources shared during Part 3: Running full text queries and combined queries with Elasticsearch and Kibana.

## Resources

[Free Elastic Cloud Trial](https://ela.st/elastic-beginners)

[Instructions](https://dev.to/elastic/downloading-elasticsearch-and-kibana-macos-linux-and-windows-1mmo) for downloading Elasticsearch and Kibana

[Presentation]()

[Dataset]() from Kaggle used for workshop

[Elastic America Virtual Chapter](https://community.elastic.co/amer-virtual/) Want to attend live workshops? Join the Elastic Vancouver Chapter to get the deets!

## Search for phrases 

### The match_phrase query
This query is used to search for terms that are near each other. 

Syntax: 
```
GET enter_the_name_of_the_index_here/_search
{
  "query": {
    "match_phrase": {
      "Specify the field you want to search over": "Enter the phrase you are searching for"
    }
  }
}
```
The following three things must happen for ingest nodes to cause a hit:
1. both "ingest" and "nodes" must appear in the content field
2. the terms must appear in that order
3. the terms must appear next to each other

Example: 
```
GET enter_the_name_of_the_index_here/_search
{
  "query": {
    "match_phrase": {
      "content": "ingest nodes"
    }
  }
}
```
Expected response from Elasticsearch:

When you search using match_phrase, you get eight results. Precision is better, but recall is much, much worse.

#### The slop Parameter 
If you want to increase the recall of a  match_phrase,  query, you can introduce some flexibility into the phrase by using the slop parameter. This allows the terms to be close to each other but not necessarily next to each other, thus increasing the recall value. The slop parameter indicates how far apart the terms are allowed to be while still considering the document a match. Take a moment to review the query below. 

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "match_phrase": {
      "Name the field you want to search over":{
        "query": "Enter the phrase you are searching for",
        "slop": "Specify number of words permitted to be between the phrases"
      }
    }
  }
}
```
Example:
```
GET name_of_index/_search
{
  "query": {
    "match_phrase": {
      "content":{
        "query": "open data",
        "slop": 1
      }
    }
  }
}
```
Allows one word to be between the search terms. 

Expected response from Elasticsearch:


#### The multi-match Query

When a user searches for something, the context of the search is not always known. You will have to make a guess. Let’s say you have a user who types in "Shay Banon" in the search bar. Is the user looking for articles by Shay Banon or articles related to Shay Banon?

There are 3 different fields you could search over for "Shay Banon." 
Author, title, and content fields. 

This type of query provides a convenient shorthand for running a match query against multiple fields. For each document, a score is computed for the term for each field. The highest of those scores will be the score for that document. This is the default scoring behavior of the multi_match query ("type": "best_field").

This type of query provides a convenient shorthand for running a match query against multiple fields. 

By default, Elasticsearch only considers the best scoring field when calculating the _score (best_fields). A score will be computed for each field and the highest score will be the one that is used to score the document. 

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"shay banon",
      "fields": [
        "title",
        "content",
        "author"
        ],
        "type": "best_fields"
    }
  }
}
````
Example:
```
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"shay banon",
      "fields": [
        "title",
        "content",
        "author"
        ],
        "type": "best_fields"
    }
  }
}
```

Expected response from Elasticsearch:

Next, take a moment to review the response from the previous query about Shay Banon below and ask yourself the following question: Are they relevant?

The answer is, yes, sort of. However, your user may not agree. Let’s try to make one field more important than the others. In our particular use case, the title field may be a good candidate. If "Shay Banon" were to be found in the title of a blog, that blog would surely be relevant to "Shay Banon."

#### Per-field boosting
Boost is one other dial to use for better search results.  Let’s say you want the blog’s title to carry more weight than the content or author fields. Review the example below to learn how you can do this.

You can boost the score of the ttile field using the carat(^) symbol
Syntax:
```
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"Enter a search phrase",
      "fields": [
        "List field you want to boost^2",
        "List field you want to search over",
        "List field you want to search over"
        ],
  }
 }
}
```
Example:
```
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"Enter a search phrase",
      "fields": [
        "title^2",
        "content",
        "author"
        ],
  }
 }
}
```
Expected response from Elasticsearch:

The response comes back with 151 hits, but the order of the hits have changed. The highest score has Shay Banon in the title, which was the field boosted. 

Is there a best practice for boosting? 

As always, it depends. Experiment and analyze your search results to find the best formula for your use case.  For our blogs dataset, search terms often appear in both the "title" and "content" fields, but searching within the title field will yield better hits.

Let's try searching for a Topic
Suppose you are looking for a blog that mentions "elasticsearch training." Notice the query here contains the popular term "elasticsearch", so naturally there will be a lot of hits. 
See google drive

Why do you think there are so many hits? 
So many hits were returned because the multi_match query performs a match query (with the default "or" logic) on multiple fields. The word "elasticsearch" occurs frequently in the Elastic blogs so naturally, there are a lot of hits.
#### Improving Precision with match_phrase
See google drive
The top hits returned are good, but the precision is not great. Let’s search for the phrase "elasticsearch training" instead. You can improve precision by performing a phrase type match (the default type is best_fields). The phrase type performs a match_phrase query on each field and selects the best score.

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"Enter search phrase",
      "fields": [
        "List field you want to boost^2",
        "List field you want to search over",
        "List field you want to search over"
        ],
        "type": "phrase"
    }
  }
}
```
Example:
```
GET news_headlines/_search
GET name_of_index/_search
{
  "query": {
    "multi_match": {
      "query":"elasticsearch training",
      "fields": [
        "title^2",
        "content",
        "author"
        ],
        "type": "phrase"
    }
  }
}
```
Expected response from Elasticsearch:

#### Mispelling

Or rather, how can you deal with "misspelled words." In today’s world, you expect a search application to grant some leniency in terms of spelling. Let’s say you search for "shark alocasion". You don’t have blogs about sharks but you do have blogs about "shard allocation".

One thing you can do to help in this situation is to add fuzziness to your queries. 
CHECK OUT NOTES ON VIDEOS from ENG I AND ADD CONTENT HERE. 
Adding fuzziness to a query
The match query has a "fuzziness" property that can be set to 0, 1, or 2. It can also be set to "auto". If you are not sure what value to set the fuzziness to, use the "auto" setting. The "auto" setting is the preferred way to use fuzziness as it defines the distance based on the length of the query terms. One thing to note, if the query has multiple terms, the fuzziness value is applied to each term separately.

Now, let's see if you can find "shard" by searching for the misspelled word "shark". Watch the demonstration below to learn more.

Fuzziness is an easy solution to the misspelling of words. However, it has a high CPU overhead in addition to your results having very low precision. Fuzziness can be used to deal with misspellings in the data, but it is not recommended for dealing with misspellings in queries. 

To learn more about how to properly handle user misspelling, it is recommended that you take the following courses:
(delve into this if I need more content)
Improving Search with Text Analysis
Improving Search with Suggestions

** Add poll questions: True or false
The match_phrase query can be used for searching for multiple terms that are near each other
The multi_match query allows you to search for the same terms in multiple fields
Fuzziness is an easy solution to deal with misspelling but has high CPU overhead and very low precision

I also like the end of the section questions. Incorporate this somehow. 

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "match": {
      "Enter the field you want to search": {
        "query": "Enter search terms",
        "fuzziness": "Enter 0,1 or 2
      }
    }
  }
}
```
Example:
```
GET name_of_index/_search
{
  "query": {
    "match": {
      "content": {
        "query": "monitering datu",
        "fuzziness": 1
      }
    }
  }
}
```
Expected response from Elasticsearch: 


