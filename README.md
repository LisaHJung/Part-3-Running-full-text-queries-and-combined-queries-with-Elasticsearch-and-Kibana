# Beginner's Crash Course to Elastic Stack Series
## Part 3: Running full text queries and combined queries with Elasticsearch and Kibana

Welcome to the Beginner's Crash Course to Elastic Stack!

This repo contains all resources shared during Part 3: Running full text queries and combined queries with Elasticsearch and Kibana.

Workshop objectives:
- delve deeper into advanced queries designed to search text fields
- build a combination of queries to answer more complex questions 
- fine tune the relevance of search results

## Resources

[Beginner's Crash Course Table of Contents](https://github.com/LisaHJung/Beginners-Crash-Course-to-the-Elastic-Stack-Series) Interested in other workshops of this series? Check out the table of contents.

[Free Elastic Cloud Trial](https://ela.st/elastic-beginners)

[Instructions](https://dev.to/elastic/downloading-elasticsearch-and-kibana-macos-linux-and-windows-1mmo) for downloading Elasticsearch and Kibana

[Presentation]()

[Dataset](https://www.kaggle.com/rmisra/news-category-dataset) from Kaggle used for workshop

[Elastic America Virtual Chapter](https://community.elastic.co/amer-virtual/) Want to attend live workshops? Join the Elastic Vancouver Chapter to get the deets!

## Review from Workshop Part 2
There are two main ways to search in Elasticsearch:
1) Queries: Queries retrieve documents that match the criteria. 
2) Aggregations: An aggregation summarizes your data as metrics, statistics, and other analytics.  

### Search queries
#### Get information about documents in an index
The following query will retrieve all documents that exist in the specified index. 

Syntax: 
```
GET enter_name_of_the_index_here/_search
```
Example: 
```
GET news_headlines/_search
```
Expected response from Elasticsearch:

Elasticsearch displays a number of hits and a sample of 10 search results by default.  

![image](https://user-images.githubusercontent.com/60980933/105432767-8c216700-5c15-11eb-9ea2-ef74a3bc5f1b.png)

### Aggregations Request
#### Analyze the data to show the categories of news headlines in our dataset
Syntax:
```
GET enter_name_of_the_index_here/_search
{
  "aggs": {
    "name your aggregation here": {
      "specify aggregation type here": {
        "field": "name the field you want to aggregate here",
        "size": state how many buckets you want returned here
      }
    }
  }
}
```
Example:
```
GET news_headlines/_search
{
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}
```
Expected response from Elasticsearch:

![image](https://user-images.githubusercontent.com/60980933/105434428-cc361900-5c18-11eb-9db7-e7441ac5a1ac.png)

## Full Text Queries
### Searching for search terms

`match query` is a standard query for performing a full text search. This query retrieves documents that contain the search terms in any way, shape or form. The order and the proximity in which the search terms are found(i.e. phrases) are not considered as a priority. 

Syntax:
```
GET enter_name_of_index_here/_search
{
  "query": {
    "match": {
      "Specify the field you want to search":{
        "query":"Enter search terms"
   }
  }
 }
}
```

###  Searching for a phrase
#### What happens when you use the `match query` to search for phrases?
Let's search for articles about Ed Sheeran's song "Shape of you" using the match query.

Example: 
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline":{
        "query":"Shape of you"
   }
  }
 }
}
```
Expected response from Elasticsearch:

![image](https://user-images.githubusercontent.com/60980933/108571746-17058800-72ce-11eb-8b44-57df1cbb35d1.png)

Elasticsearch returns greater than 10,000 hits. The top hit as well as many others in the search results only contain the search terms "you" and "shape". These terms are not found in the same order or proximity to each other as the search query "Shape of you".  Along with a few articles about the song "Shape of you", it pulls up articles about being in shape or what shape of your face says about you. 

When the `match query` is used to search for a phrase, it has high recall but low precision as it returns a lot of loosely related documents.

#### Searching for phrases using the `match_phrase` query
Syntax: 
```
GET enter_name_of_index_here/_search
{
  "query": {
    "match_phrase": {
      "Specify the field you want to search":{
        "query":"Enter search terms"
   }
  }
 }
}
```
Example: 
```
GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline":{
        "query":"Shape of You"
   }
  }
 }
}
```
When the match_phrase parameter is used, all hits returned must meet the following criteria:
1. the search terms "Shape", "of", and "you" must appear in the headline field
2. the terms must appear in that order
3. the terms must appear next to each other

Expected response from Elasticsearch:

With match_phrase parameter, we get 3 hits returned. All 3 hits satisfy the criteria mentiond above. 

![image](https://user-images.githubusercontent.com/60980933/108422566-f537d280-71f3-11eb-8340-2899b5fbc61e.png)

With the match_phrase parameter, you get higher precision but lower recall. 

### Running a match query against multiple fields 

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
        ]
    }
  }
}
````
Example:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query":"Michelle Obama",
      "fields": [
        "headline",
        "short_description",
        "author"
        ]
    }
  }
}

```
Expected response from Elasticsearch:

We see 3044 hits that contain "Michelle Obama" in the headline or short_description or author field. But you will see that this hit came up even though Michelle Obama was not the main topic of the article. 

![image](https://user-images.githubusercontent.com/60980933/108424827-e9014480-71f6-11eb-90f5-f7276aebbd0d.png)

#### Per-field boosting
What if you want the article's headline to carry more weight than short_description or author fields? 

You can boost the score of the headline field using the carat(^) symbol.
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
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query":"michelle obama",
      "fields": [
        "headline^2",
        "short_description",
        "author"
        ]
    }
  }
}
```
Expected response from Elasticsearch:

![image](https://user-images.githubusercontent.com/60980933/108425052-2fef3a00-71f7-11eb-8bf1-8c13d6a0a37c.png)

Yields same number of hits(3044) but the order of the hits have changed. The hits ranking higher on the list has Michelle Obama in the boosted field, headline. These articles are definitely about Michelle Obama!

#### Let's try searching for a Topic
While searching for Michelle Obama, the user just remembered she is throwing a party for all of her friends this weekend. She searches for articles regarding party planning to get some ideas for it. 

```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query":"party planning",
      "fields": [
        "headline^2",
        "short_description"
        ]
    }
  }
}
```
Response from Elasticsearch:
This query yields a lot of hits(2846). This happens because multi_match query performs a match query in multiple fields. The terms "party" or "planning" are popular terms. If any one of these search terms appear in any of these fields in any way shape or form, the document will be considered as a hit. Because of that, you will see irrelevant search results included among the hits as well. 

![image](https://user-images.githubusercontent.com/60980933/108582689-09fa9000-72f2-11eb-8e34-8f6cc4302254.png)

#### Improving Precision with phrase type match. 
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
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query":"party planning",
      "fields": [
        "headline^2",
        "short_description"
        ],
        "type": "phrase"
    }
  }
}
```
Expected response from Elasticsearch:

The recall is much lower(6 hits vs 2846) but every one of the hits have the phrase party planning in either headline or short_description field. The hits that have phrase party planning in boosted field headline is ranked higher in the search results. 

![image](https://user-images.githubusercontent.com/60980933/108582734-4ded9500-72f2-11eb-856b-611cc81e11bf.png)


### Combined Queries

While 

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
Find what categories Michelle Obama is found. 
![image](https://user-images.githubusercontent.com/60980933/108541130-5668b000-729f-11eb-80aa-8e37b6dc016c.png)
```
GET news_headlines/_search
{
  "query": {
    "match_phrase": { 
      "headline": "Michelle Obama"
   }
  },
  "aggregations": {
    "categories_found": {
      "terms": {
       "field": "category",
       "size": 100
   }
  }
 }
}
```
#### The must Clause
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
         }
        },
        {
          "match": {
            "category": "POLITICS"
          }
        }
      ]
    }
  }
}
```
Expected response from Elasticsearch: 
![image](https://user-images.githubusercontent.com/60980933/108541719-0807e100-72a0-11eb-8872-4fa3d121129b.png)
See Google doc
Let’s take a look at the must clause. The must clause requires that "anything in here must match." It’s almost like an "and" operator. The queries in the must clause must match in the returned documents. Each query will compute its own score separately and will contribute to the overall score. 

Elastic writes a lot of blogs about product releases, which may not be relevant if you are looking for technical details. Let’s search for "Logstash" only in the "Engineering" category.

With these types of queries, Elasticsearch will search across two fields. Both match queries must be satisfied for a document to be returned as a hit. This means that having more queries in your must clause will increase the precision of your query. The score will be computed for both clauses and then added together. 

#### The must_not Clause
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": {
        "match_phrase": {
          "headline": "Michelle Obama"
         }
        },
        "must_not":[
         {
          "match_phrase": {
            "category": "WEDDINGS"
          }
        }
      ]
    }
  }
}
```
Expected:
![image](https://user-images.githubusercontent.com/60980933/108543691-a137f700-72a2-11eb-93d1-2dfae56096d8.png)

See google doc
The must_not clause is used to exclude information. The clause (query) must not appear in the matching documents. Since the term does not appear in the documents, all returned documents have a score of 0. In fact, the scoring algorithm is skipped altogether.

The previous query for "Logstash" and "Engineering" cast a narrow net; it is too precise. You may see better results, with more recall, by  searching for blogs not in the category "Releases."

The previous query does not cast a wide net for "Logstash." It might be better not to search for blogs in the category "Releases."

#### The should Clause

```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
          }
         }
        ],
        "should":[
         {
          "match_phrase": {
            "category": "BLACK VOICES"
          }
        }
      ]
    }
  }
}
```
Expected response: 
![image](https://user-images.githubusercontent.com/60980933/108544983-7058c180-72a4-11eb-8adc-0433592ef9a9.png)
Black history month- when people search for Michelle obama, more likely to be related to the category Black Voices rather than parenting or style.  perhaps we should prioritize

A should clause does not exclude documents. The queries in the should clause should appear in the matching document, and if it does, the score of that document will increase so that the document will return earlier in hits. 

Let’s say you are looking for blogs about the "Elastic Stack" preferably from "Shay Banon." The query will return all blogs about the Elastic Stack, and those documents authored by Shay Banon will come back first.

This should clause does not add more hits, but blogs written by Shay score the highest.

#### Filter Clause
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
          }
         }
        ],
        "filter":{
          "range":{
             "date": {
               "lt": "2016-03-25"
          }
        }
      }
    }
  }
}
```

The filter clause will exclude some results, so does it actually match? Filter clauses should be a yes/no question if you want to use this clause and no scores will be calculated.

Let’s say you want to find blogs about the "Elastic Stack" published before 2017.

You want to place your range query on the publish date field into a filter clause. Dates are either in (yes) or not in (no) any date range. Run the range query on its own and you’ll find that all the documents that are returned have a score of 1 (yes). This score was actually computed for every document that matched, which is a waste of computation time. Place queries like this range query in a filter clause and the scoring algorithm will be skipped for that query.

The filter clause will strip away any hits that do not match the filter.
Expected: 
![image](https://user-images.githubusercontent.com/60980933/108546105-e6a9f380-72a5-11eb-8983-f4718e95140e.png)

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


#### Improving relevance 

You can control much of precision and scoring within your bool queries. The following section contains tips for how to control and improve the precision of your hits as well as some clever uses of the should clause to cast a wide net and also have high precision by making use of rankings. 

many should clauses
Suppose you run a search for a blogs with “Elastic” in the title field and you want to favor blogs that mention “stack,” “speed,” or “query.” Let’s add a few queries in the should clause.  Review the example below. 

This will cast a wide net because none of the queries in the should clause need to match. 
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_phrase": {"headline": "Michelle Obama"}}
        ],
        "should":[
          {"match": {"headline": "Becoming"}},
          {"match": {"headline": "women"}},
          {"match": {"headline": "empower"}}
      ]
    }
  }
}
```
Expected:
![image](https://user-images.githubusercontent.com/60980933/108548611-51a8f980-72a9-11eb-8310-0fe14286e437.png)

use minimum_should_match
–
You can also improve relevance by having multiple should clauses together with a minimum_should_match parameter. With this optional parameter, you can require that one or more queries in the should clause match. Let’s improve precision by requiring at least one of the three queries in the should clause to match.



This query returns documents that match the query in the must clause and at least one of the queries in the should clause.
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_phrase": {"headline": "Michelle Obama"}}
        ],
        "should":[
          {"match": {"headline": "Becoming"}},
          {"match": {"headline": "women"}},
          {"match": {"headline": "empower"}}
      ],
      "minimum_should_match": 1
    }
  }
}
```

Expected: 
![image](https://user-images.githubusercontent.com/60980933/108549645-aac55d00-72aa-11eb-9337-e49bb46531c2.png)
![image](https://user-images.githubusercontent.com/60980933/108549665-b44ec500-72aa-11eb-9c33-bd186827b110.png)

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
