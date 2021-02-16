# Beginner's Crash Course to Elastic Stack Series
## Part 3: Running full text queries and combined queries with Elasticsearch and Kibana

Welcome to the Beginner's Crash Course to Elastic Stack!

This repo contains all resources shared during Part 3: Running full text queries and combined queries with Elasticsearch and Kibana.

Workshop objectives:
- delve deeper into advanced search queries designed to search text fields
- build a combination of queries to answer more complex questions 
- fine tune relevancy of complex queries

## Resources

[Beginner's Crash Course Table of Contents]() Interested in other workshops of this series? Check out the table of contents.

[Free Elastic Cloud Trial](https://ela.st/elastic-beginners)

[Instructions](https://dev.to/elastic/downloading-elasticsearch-and-kibana-macos-linux-and-windows-1mmo) for downloading Elasticsearch and Kibana

[Presentation]()

[Dataset]() from Kaggle used for workshop

[Elastic America Virtual Chapter](https://community.elastic.co/amer-virtual/) Want to attend live workshops? Join the Elastic Vancouver Chapter to get the deets!

## Get information about documents in an index

Syntax: 
```
GET enter_name_of_the_index_here/_search
```
Example: 
```
GET song_lyrics/_search
```
Expected response from Elasticsearch:

Elasticsearch displays a number of hits and a sample of 10 search results by default.  

![image](https://user-images.githubusercontent.com/60980933/107825991-53bb0780-6d41-11eb-90f9-eef0b5d0004f.png)

## Search for phrases 
The match_phrase query is used to search for phrases(i.e. a group of search terms that appear in that order and are next to each other). 

Syntax: 
```
GET Enter_the_name_of_the_index_here/_search
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
GET index/_search
{
  "query": {
    "match_phrase": {
      "Lyrics": "ingest nodes" or "open data"
    }
  }
}
```

Expected response from Elasticsearch:

When you search using match_phrase, you get eight results. Precision is better, but recall is much, much worse.

#### The slop Parameter 
If you want to increase the recall of a  match_phrase query, you can introduce some flexibility into the phrase by using the slop parameter. This allows the terms to be close to each other but not necessarily next to each other, thus increasing the recall value. The slop parameter indicates how far apart the terms are allowed to be while still considering the document a match. Take a moment to review the query below. 

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
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

#### Running a match query against multiple fields 

When a user searches for something, the context of the search is not always known. You will have to make a guess. Let’s say you have a user who types in "Shay Banon" in the search bar. Is the user looking for articles by Shay Banon or articles related to Shay Banon?

There are 3 different fields you could search over for "Shay Banon." 
Author, title, and content fields. 

Why not query all three of those fields? You can query for Shay Banon in the author, title, and content fields using a multi_match query. 

The multi-match query provides a convenient shorthand for running a match query against multiple fields. For each document, a score is computed for the term for each field. The highest of those scores will be the score for that document. This is the default scoring behavior of the multi_match query ("type": "best_field").

This type of query provides a convenient shorthand for running a match query against multiple fields. 

By default, Elasticsearch only considers the best scoring field when calculating the _score (best_fields). A score will be computed for each field and the highest score will be the one that is used to score the document. 

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
{
  "query": {
    "multi_match": {
      "query":"Enter search terms here",
      "fields": [
        "List the field you want to search over",
        "List the field you want to search over",
        "List the field you want to search over"
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

You can boost the score of the title field using the carat(^) symbol.
Syntax:
```
GET Enter_the_name_of_the_index_here/_search
{
  "query": {
    "multi_match": {
      "query":"Enter search terms",
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

#### Let's try searching for a Topic
Suppose you are looking for a blog that mentions "elasticsearch training." Notice the query here contains the popular term "elasticsearch", so naturally there will be a lot of hits. 
See google drive

Why do you think there are so many hits? 
So many hits were returned because the multi_match query performs a match query (with the default "or" logic) on multiple fields. The word "elasticsearch" occurs frequently in the Elastic blogs so naturally, there are a lot of hits.

#### Improving Precision with match_phrase
See google drive
The top hits returned are good, but the precision is not great. Let’s search for the phrase "elasticsearch training" instead. You can improve precision by performing a phrase type match (the default type is best_fields). The phrase type performs a match_phrase query on each field and selects the best score.

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
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

### Combined Queries

Suppose you want to write a query to answer the following request: 

"Find blogs about the "Elastic Stack" published before 2017"  

This search is actually a combination of two queries:

Search for "Elastic Stack" in the content or title field
The publish_date field needs to be before 2017
You can combine these two queries by using the bool query.

#### Bool Query
A bool query is a query that matches documents matching boolean combinations of other queries. There are four clauses to choose from: must, must_not, should, and filter. You can build combinations of one or more queries by including them in one or more of these clauses. You will take a look at each clause to understand the differences between them. 

Each of the following clauses are possible (but optional) in a bool query and they can appear in any order. The order doesn't matter because the clauses will be internally reordered based on their respective costs. Take a moment to review the syntax below.

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "bool": {
      "must": [
        {}
      ],
      "must_not": [
        {}
      ],
      "should": [
        {}
      ],
      "filter": [
        {}
      ]
    }
  }
}
```
#### The must Clause
See Google doc
Let’s take a look at the must clause. The must clause requires that "anything in here must match." It’s almost like an "and" operator. The queries in the must clause must match in the returned documents. Each query will compute its own score separately and will contribute to the overall score. 

Elastic writes a lot of blogs about product releases, which may not be relevant if you are looking for technical details. Let’s search for "Logstash" only in the "Engineering" category.

With these types of queries, Elasticsearch will search across two fields. Both match queries must be satisfied for a document to be returned as a hit. This means that having more queries in your must clause will increase the precision of your query. The score will be computed for both clauses and then added together. 

#### The must_not Clause

See google doc
The must_not clause is used to exclude information. The clause (query) must not appear in the matching documents. Since the term does not appear in the documents, all returned documents have a score of 0. In fact, the scoring algorithm is skipped altogether.

The previous query for "Logstash" and "Engineering" cast a narrow net; it is too precise. You may see better results, with more recall, by  searching for blogs not in the category "Releases."

The previous query does not cast a wide net for "Logstash." It might be better not to search for blogs in the category "Releases."

#### The should Clause
A should clause does not exclude documents. The queries in the should clause should appear in the matching document, and if it does, the score of that document will increase so that the document will return earlier in hits. 

Let’s say you are looking for blogs about the "Elastic Stack" preferably from "Shay Banon." The query will return all blogs about the Elastic Stack, and those documents authored by Shay Banon will come back first.

This should clause does not add more hits, but blogs written by Shay score the highest.

#### Query vs Filter
You have seen query contexts, but there is another clause known as a filter context:

Query context: results in a _score
Filter context: results in a "yes" or "no"
If you want to know how well the document matches the query, use the query context. If you don’t care about the score and only want to know if the document matches the query, use the filter context.

Queries that are placed in a filter clause or must_not clause are in a filter context. This means that scoring is skipped, and also become candidates for caching. You can learn more about caching in module 3 of Elasticsearch Engineer II. 

see Google doc

#### The bool Query: Clauses Recap

To recap, must, must_not, and filter clauses all have the ability to include or exclude documents to be part of the result set. Should, on the other hand, does not (by default) include or exclude documents from the result set. There are however some exceptions to this rule, and you will learn about this later on.

The filter and must_not do not contribute to the score of the documents while must and should do.

#### The filter clause
The filter clause will exclude some results, so does it actually match? Filter clauses should be a yes/no question if you want to use this clause and no scores will be calculated.

Let’s say you want to find blogs about the "Elastic Stack" published before 2017.

You want to place your range query on the publish date field into a filter clause. Dates are either in (yes) or not in (no) any date range. Run the range query on its own and you’ll find that all the documents that are returned have a score of 1 (yes). This score was actually computed for every document that matched, which is a waste of computation time. Place queries like this range query in a filter clause and the scoring algorithm will be skipped for that query.

The filter clause will strip away any hits that do not match the filter.

#### Improving relevance 

You can control much of precision and scoring within your bool queries. The following section contains tips for how to control and improve the precision of your hits as well as some clever uses of the should clause to cast a wide net and also have high precision by making use of rankings. 

many should clauses
Suppose you run a search for a blogs with “Elastic” in the title field and you want to favor blogs that mention “stack,” “speed,” or “query.” Let’s add a few queries in the should clause.  Review the example below. 

This will cast a wide net because none of the queries in the should clause need to match. 

use minimum_should_match
–
You can also improve relevance by having multiple should clauses together with a minimum_should_match parameter. With this optional parameter, you can require that one or more queries in the should clause match. Let’s improve precision by requiring at least one of the three queries in the should clause to match.



This query returns documents that match the query in the must clause and at least one of the queries in the should clause.

only should clauses
If you have a bool query with no queries in the must or filter clause, one or more queries in the should clause must match. This means that the minimum_should_match defaults to 1 when your query does not have a must or filter clause. (If your query does have a must or filter clause, minimum_should_match defaults to 0).

"should not" query

There is no built-in "should not" clause. But you can still build a query that negatively impacts the score by combining bool queries. The following example implements a “should not” logic. Notice that bool queries can be nested. 



   “Find blogs that contain “elastic”, but I prefer blogs that do not contain “stack.” 



You cast a wider net here by not filtering out “stack”, but you improve the precision by adding to the scores of the documents that do not contain “stack”. 

A search tip for phrases
Let's recall your search for “open data” in the previous lesson. 

Take a moment to review the example below. Notice there are many hits because "open" and "data" are common terms.

Another search tip for phrases
You can improve the precision of this query by using a match_phrase query. You could also use an “and” operator instead of the default “or” operator to improve the precision. However, both of these options narrow your search results and your recall goes down.



In the example below, you keep the high recall of the match query using the “or” logic and also improve the precision by reordering the results to return the high precision documents first. Notice you get the same 852 hits, but the score is higher for documents in which the phrase "open data" appears.

- Add other ways to query Elasticsearch, KQL, and SQL if I need more content
