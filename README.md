# Beginner's Crash Course to Elastic Stack Series
## Part 3: Running full text queries and combined queries with Elasticsearch and Kibana

Welcome to the Beginner's Crash Course to Elastic Stack!

This repo contains all resources shared during Part 3: Running full text queries and combined queries with Elasticsearch and Kibana.

Workshop objectives:
- delve deeper into advanced queries designed to search text fields
- build a combination of queries to answer more complex questions 
- fine-tune the relevance of search results

## Resources

[Beginner's Crash Course to Elastic Stack Table of Contents](https://github.com/LisaHJung/Beginners-Crash-Course-to-the-Elastic-Stack-Series) This workshop is part 3 of the Beginner's Crash Course to Elastic Stack series. Check out this table contents to access all the workshops in the series thus far. This table will continue to get updated as more workshops in the series are released! 

[Free Elastic Cloud Trial](https://www.elastic.co/cloud/cloud-trial-overview/30-days?ultron=community-beginners-crash-course-May2022+&hulk=30d)

[Instructions](https://dev.to/lisahjung/beginner-s-guide-to-setting-up-elasticsearch-and-kibana-with-elastic-cloud-1joh) on how to access Elasticsearch and Kibana on Elastic Cloud

[Instructions](https://dev.to/elastic/downloading-elasticsearch-and-kibana-macos-linux-and-windows-1mmo) for downloading Elasticsearch and Kibana

[Video Recording](https://www.youtube.com/watch?v=2KgJ6TQPIIA&t=374s) of the workshop

[Mini Beginner's Crash Course to Elasticsearch & Kibana playlist](https://ela.st/mini-beginners-crash-course)

Do you prefer learning by watching shorter videos? Check out this playlist to watch short clips of beginner's crash course full length workshops. Part 3 workshop is broken down into episodes 11-14.  Season 2 clips will be uploaded here in the future! 

[Presentation](https://github.com/LisaHJung/Part-3-Running-full-text-queries-and-combined-queries-with-Elasticsearch-and-Kibana/blob/main/Part%203_%20Running%20full%20text%20queries%20and%20combined%20queries%20with%20Elasticsearch%20and%20Kibana.pdf)

[News headlines dataset](https://www.kaggle.com/rmisra/news-category-dataset) from Kaggle used for workshop

[What's next?](https://github.com/LisaHJung/Part-4-Running-Aggregations-with-Elasticsearch-and-Kibana) Eager to continue your learning after mastering the concept from this workshop? Move on to Part 4: Running Aggregations with Elasticsearch and Kibana [here](https://github.com/LisaHJung/Part-4-Running-Aggregations-with-Elasticsearch-and-Kibana)!

## Review from Workshop Part 2
There are two main ways to search in Elasticsearch:
1) `Queries` retrieve documents that match the specified criteria. 
2) `Aggregations` present the summary of your data as metrics, statistics, and other analytics.  

### Search queries
#### Get information about documents in an index
The following `query` will retrieve all documents that exist in the specified index. This `query` is a great way to explore the structure and content of your document. 

Syntax: 
```
GET Enter_name_of_the_index_here/_search
```
Example: 
```
GET news_headlines/_search
```
Expected response from Elasticsearch:

Elasticsearch displays a number of hits and a sample of 10 search results by default. The field "_ source"(line 22) lists all fields(the content) of a document.

![image](https://user-images.githubusercontent.com/60980933/105432767-8c216700-5c15-11eb-9ea2-ef74a3bc5f1b.png)

### Aggregations Request
#### Analyze the data to show the categories of news headlines in our dataset
Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "aggregations": {
    "Name your aggregation here": {
      "Specify aggregation type here": {
        "field": "Name the field you want to aggregate here",
        "size": State how many buckets you want returned here
      }
    }
  }
}
```
Example:
```
GET news_headlines/_search
{
  "aggregations": {
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

When you minimize hits(line 10), you will see the aggregations report we named by_category. This report displays all categories that exist in our datset as well as the number of documents that fall under each category. 

![image](https://user-images.githubusercontent.com/60980933/105434428-cc361900-5c18-11eb-9db7-e7441ac5a1ac.png)

## Full Text Queries
### Searching for search terms

The `match query` is a standard query for performing a full text search. This `query` retrieves documents that contain the search terms. It uses "OR" logic by default, meaning that it will retrieve documents that contain any one of the search terms. The order and the proximity in which the search terms are found(i.e. phrases) are not taken into account.

Syntax:
```
GET Enter_name_of_index_here/_search
{
  "query": {
    "match": {
      "Specify the field you want to search": {
        "query": "Enter search terms"
      }
    }
  }
}
```
![image](https://user-images.githubusercontent.com/60980933/135136014-0c608cfe-9991-459b-a749-505f7c699fcf.png)

###  Searching for a phrase
#### What happens when you use the `match query` to search for phrases?
Let's search for articles about Ed Sheeran's song "Shape of you" using the `match query`.

Example: 
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Shape of you"
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch returns greater than 10,000 hits. The top hit as well as many others in the search results only contain the search terms "you" and "shape". These terms are not found in the same order or in proximity to each other as the search terms "Shape of you". 

When the `match query` is used to search for a phrase, it has high recall but low precision as it returns a lot of loosely related documents.

![image](https://user-images.githubusercontent.com/60980933/108571746-17058800-72ce-11eb-8b44-57df1cbb35d1.png)

Along with a few articles about the song "Shape of you", it pulls up articles about being in shape or what shape of your face says about you. 

When the `match query` is used to search for a phrase, it has high recall but low precision. It pulls up more loosely related documents as it uses "OR" logic by default. It pulls up documents that contains any one of the search terms in the specified field. Moreover, the order and the proximity in which the search terms are found are not taken into account.

#### Searching for phrases using the `match_phrase` query

If the order and the proximity in which the search terms are found(i.e. phrases) are important in determining the relevance of your search, you use the `match_phrase query`.

Syntax: 
```
GET Enter_name_of_index_here/_search
{
  "query": {
    "match_phrase": {
      "Specify the field you want to search": {
        "query": "Enter search terms"
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
      "headline": {
        "query": "Shape of You"
      }
    }
  }
}
```
When the `match_phrase` parameter is used, all hits must meet the following criteria:
1. the search terms "Shape", "of", and "you" must appear in the field headline .
2. the terms must appear in that order.
3. the terms must appear next to each other.

Expected response from Elasticsearch:

With `match_phrase` parameter, we get 3 hits returned. All 3 hits satisfy the criteria mentioned above. 

The `match_phrase` parameter yields higher precision but lower recall as it takes the order and the proximity in which the search terms are found into account. 

![image](https://user-images.githubusercontent.com/60980933/108422566-f537d280-71f3-11eb-8340-2899b5fbc61e.png)

### Running a match query against multiple fields 

When designing a `query`, you don't always know the context of a user's search. When a user searches for "Michelle Obama", the user could be searching for statements written by Michelle Obama or articles written about her.

To accommodate these contexts, you can write a `multi_match query`, which searches for terms in multiple fields.

The `multi_match query` runs a `match query` on multiple fields and calculates a score for each field. Then, it assigns the highest score among the fields to the document.

This score will determine the ranking of the document within the search results.

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
{
  "query": {
    "multi_match": {
      "query": "Enter search terms here",
      "fields": [
        "List the field you want to search over",
        "List the field you want to search over",
        "List the field you want to search over"
      ]
    }
  }
}
````
The following `multi_match query` asks Elasticsearch to `query` documents that contain the search terms "Michelle" or "Obama" in the fields headline, or short_description, or authors.

Example:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline",
        "short_description",
        "authors"
      ]
    }
  }
}
```
Expected response from Elasticsearch:

We see 3044 hits that contain "Michelle Obama" in the headline or short_description or author field. While the `multi_match query` increased the recall, it decreased the precision of the hits. 

For example, in our search for "Michelle Obama" related headlines, the top hit is a news headline featuring Bernie Sanders as the main topic. In this headline, Michelle Obama is mentioned once in the field short_description.

![image](https://user-images.githubusercontent.com/60980933/127704425-c1ade381-dff4-446f-9980-1caa9a85e972.png)

#### Per-field boosting
Headlines mentioning "Michelle Obama" in the field headline are more likely to be related to our search than the headlines that mention "Michelle Obama" once or twice in the field short_description.

To improve the precision of your search, you can designate one field to carry more weight than the others.

This can be done by boosting the score of the field headline(`per-field boosting`). This is notated by adding a carat(^) symbol and number 2 to the desired field as shown below.

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
{
  "query": {
    "multi_match": {
      "query": "Enter search terms",
      "fields": [
        "List field you want to boost^2",
        "List field you want to search over",
        "List field you want to search over"
      ]
    }
  }
}
```
The following example boosts the score of documents that contain the search terms in the field headline. If the term "Michelle" or "Obama" are found in the field headline of a document, that document is given a higher score and is ranked higher in the search results.

Example:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline^2",
        "short_description",
        "authors"
      ]
    }
  }
}
```
Expected response from Elasticsearch:

`Per-field boosting` yields same number of hits(5128). However, it changes the ranking of the hits. The hits ranked higher on the list contains the search terms "Michelle Obama" in the boosted field, headline.

![image](https://user-images.githubusercontent.com/60980933/108877609-a5268a80-75bc-11eb-8f61-ce1d1702a6e0.png)

The documents containing the search terms "Michelle Obama" in the field headline are more likely to be about Michelle Obama. By using the `per-field boosting`, we have improved the precision of our search!

#### What happens when you use the `multi_match` query to search for a phrase?

While searching for Michelle Obama, the user remembers that she is throwing a party for all of her friends this weekend. She searches for news headlines regarding "party planning" to get some ideas for it.

She uses the `multi_match query` to search for the phrase party planning.

Example:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "party planning",
      "fields": [
        "headline^2",
        "short_description"
      ]
    }
  }
}
```
Response from Elasticsearch:

This query yields a lot of hits(2846). 

![image](https://user-images.githubusercontent.com/60980933/108582689-09fa9000-72f2-11eb-8e34-8f6cc4302254.png)

But why does one of our top 10 hits feature Bernie Sanders **planning** a national tour for grassroots **party** activism?

The terms "party" and "planning" are popular terms found in many documents.

With the `multi_match query`, a document is considered as a hit if any one of these search terms were found in any one of the specified fields. It does not take into account the order or the proximity in which these search terms are found.

Because of that, you will see loosely related search results included among the top hits.

#### Improving precision with phrase type match
 
You can improve the precision of a `multi_match query` by adding the "type":"phrase" to the query.

The phrase type performs a `match_phrase query` on each field and calculates a score for each field. Then, it assigns the highest score among the fields to the document.

Syntax:
```
GET Enter_the_name_of_the_index_here/_search
{
  "query": {
    "multi_match": {
      "query": "Enter search phrase",
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
The following query asks Elasticsearch to look up the phrase "party planning" in the fields headline and short_description.

Using `per field boosting`, this query assigns a higher score to documents containing the phrase "party planning" in the field headline. The documents that include the phrase "party planning" in the field headline will be ranked higher in the search results.

Example:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "party planning",
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

The recall is much lower(6 vs 2846 hits) but every one of the hits have the phrase "party planning" in either the field headline or short_description or both.

![image](https://user-images.githubusercontent.com/60980933/108615437-4943e200-73c1-11eb-87ac-881b6a20d962.png)

Among these, the hits that have the phrase "party planning" in the boosted field headline are ranked higher in the search results and presented towards the top of the search results.

## Combined Queries

There will be times when a user asks a multi-faceted question that requires multiple `queries` to answer.

For example, a user may want to find political headlines about Michelle Obama published before the year 2016.

This search is actually a combination of three `queries`:

1) Query headlines that contain the search terms "Michelle Obama" in the field headline.
2) Query "Michelle Obama" headlines from the "POLITICS" category.
3) Query "Michelle Obama" headlines published before the year 2016

One of the ways you can combine these queries is through a `bool query`.

### Bool Query
The [bool query](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/query-dsl-bool-query.html#:~:text=Bool%20Queryedit,clause%20with%20a%20typed%20occurrence.) retrieves documents matching boolean combinations of other `queries`. 

With the `bool query`, you can combine multiple `queries` into one request and further specify boolean clauses to narrow down your search results.

There are four clauses to choose from: 

1. must
2. must_not
3. should 
4. filter

You can build combinations of one or more of these clauses.
Each clause can contain one or multiple queries that specify the criteria of each clause. 

These clauses are optional and can be mixed and matched to cater to your use case. The order in which they appear does not matter either! 

Syntax:
```
GET name_of_index/_search
{
  "query": {
    "bool": {
      "must": [
        {One or more queries can be specified here. A document MUST match all of these queries to be considered as a hit.}
      ],
      "must_not": [
        {A document must NOT match any of the queries specified here. It it does, it is excluded from the search results.}
      ],
      "should": [
        {A document does not have to match any queries specified here. However, it if it does match, this document is given a higher score.}
      ],
      "filter": [
        {These filters(queries) place documents in either yes or no category. Ones that fall into the yes category are included in the hits. }
      ]
    }
  }
}
```
#### A combination of query and aggregation request

A `bool query` can help you answer multi-faceted questions. Before we go over the four clauses of the bool query, we need to first understand what type of questions we can ask about Michelle Obama.

Let's first figure out what headlines have been written about her.

One way to understand that is by searching for categories of headlines that mention Michelle Obama.

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "Enter match or match_phrase here": { "Enter the name of the field": "Enter the value you are looking for" }
  },
  "aggregations": {
    "Name your aggregation here": {
      "Specify aggregation type here": {
        "field": "Name the field you want to aggregate here",
        "size": State how many buckets you want returned here
      }
    }
  }
}
```
The following query ask Elasticsearch to query all data that has the phrase "Michelle Obama" in the headline. Then, perform aggregations on the queried data and retrieve up to 100 categories that exist in the queried data.

Example:
```
GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline": "Michelle Obama"
    }
  },
  "aggregations": {
    "category_mentions": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}
```
Expected reponse from Elasticsearch:

When you minimize the hits field(line 10), you will see an aggregations report called category_mentions. This report displays an array of all the categories that exist in the queried data and the number of headlines that have been written about each category.

We see that many news headlines about Michelle Obama has been written under categories such as "POLITICS", "BLACK VOICES", "PARENTING", "TASTE", and even "WEDDINGS"!

![image](https://user-images.githubusercontent.com/60980933/108541130-5668b000-729f-11eb-80aa-8e37b6dc016c.png)

**Now let's get back to the bool query!**

With the `bool query`, you can combine multiple `queries` into one request and further specify boolean clauses to narrow down your search results.

There are four clauses to choose from:

- must
- must_not
- should
- filter

#### The must clause
The `must clause` defines all `queries`(criteria) a document MUST match to be returned as hits. These criteria are expressed in the form of one or multiple `queries`.

All `queries` in the `must clause` must be satisfied for a document to be returned as a hit. As a result, having more `queries` in the `must clause` will increase the precision of your `query`. 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Enter match or match_phrase here": {
          "Enter the name of the field": "Enter the value you are looking for" 
         }
        },
        {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for" 
          }
        }
      ]
    }
  }
}
```
The following is a `bool query` that uses the `must clause`. This `query` specifies that all hits must match the phrase "Michelle Obama" in the field headline and match the term "POLITICS" in the field category.

Example:
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

You get 45 hits. All documents contain the phrase "Michelle Obama" in the field headline and the term "POLITICS" in the field category.

![image](https://user-images.githubusercontent.com/60980933/108631596-cbff8800-7427-11eb-9ac0-2f172075f9ed.png)

#### The must_not clause
The `must_not` clause defines `queries`(criteria) a document MUST NOT match to be included in the search results. 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Enter match or match_phrase here": {
          "Enter the name of the field": "Enter the value you are looking for" 
         }
        },
       "must_not":[
         {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        }
      ]
    }
  }
}
```
What if you want all Michelle Obama headlines except for the ones that belong in the "WEDDINGS" category?

The following `bool query` specifies that all hits must contain the phrase "Michelle Obama" in the field headline. However, the hits `must_not` contain the term "WEDDINGS" in the field category.

Example:

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
          "match": {
            "category": "WEDDINGS"
          }
        }
      ]
    }
  }
}
```
Expected response from Elasticsearch:

This `query` increases the recall(203 hits). It pulls up all the hits that contain the phrase "Michelle Obama" in the field headline. Among the hits, Elasticsearch excludes all documents that contain the term "WEDDINGS" in the field category.

![image](https://user-images.githubusercontent.com/60980933/108631753-99a25a80-7428-11eb-819a-c284e0ebb1b1.png)

#### The should clause
The `should clause` adds "nice to have" `queries`(criteria). The documents do not need to match the "nice to have" `queries` to be considered as hits. However, the ones that do will be given a higher score so it shows up higher in the search results. 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Enter match or match_phrase here: {
          "Enter the name of the field": "Enter the value you are looking for" 
         }
        },
       "should":[
         {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        }
      ]
    }
  }
```

Let's talk about a scenario where we may use the `should clause`. During the Black History Month, it is possible that the user may be looking up "Michelle Obama" in the context of "BLACK VOICES" category rather than in the context of "WEDDINGS", "TASTE", or "STYLE" categories.

To accommodate this scenario, you may write a `query` where all hits MUST contain "Michelle Obama" in the field headline. Having the phrase "BLACK VOICES" in the category is not required. However, if a document contains the phrase "BLACK VOICES" in the field category, this document should be given a higher score and should be placed higher in the search results.

To accommodate this scenario, you would write the following `bool query`. It specifies that all hits must match the phrase "Michelle Obama" in the field headline. `Should` a hit match the phrase "BLACK VOICES" in the field category, this hit will be given a higher score and will be shown higher in the search results.

Example:
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

We should still get the same number of hits(207) as the `should clause` does not add or exclude more hits. However, you will notice that the ranking of the documents has been changed. The documents with the phrase "BLACK VOICES" in the field category are now presented at the top of the search results.

![image](https://user-images.githubusercontent.com/60980933/108632370-be4c0180-742b-11eb-8c03-75c7809b54b8.png)

#### The filter clause
The `filter clause` contains filter `queries` that place documents into either "yes" or "no" category.

For example, let's say you are looking for headlines published within a certain time range. Some documents will fall within this range(yes) or do not fall within this range(no).

The `filter clause` only includes documents that fall into the yes category.

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Enter match or match_phrase here": {
          "Enter the name of the field": "Enter the value you are looking for" 
         }
        }
        ],
       "filter":{
          "range":{
             "date": {
               "gte": "Enter lowest value of the range here",
               "lte": "Enter highest value of the range here"
          }
        }
      }
    }
  }
}
```
Let's say we wanted to retrieve hits that must include the phrase "Michelle Obama" in the field headline. Among these hits, you want to include documents published within the date range "2014-03-25" and "2016-03-25".

You `bool query` will look something like this.

Example:
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
               "gte": "2014-03-25",
               "lte": "2016-03-25"
          }
        }
      }
    }
  }
}
```
Expected response from Elasticsearch: 

You will see 33 hits returned. All hits contain the phrase "Michelle Obama" in the field headline. All hits were published between the date range we specified under the `filter clause`.  

![image](https://user-images.githubusercontent.com/60980933/108633174-15ec6c00-7430-11eb-9153-05d849673f3a.png)

All right. Now that we have mastered the `bool query`, let's figure out how we can fine-tune the relevance of `bool queries`!

#### Fine-tuning the relevance of bool queries

There are many ways you can fine-tune the relevance of `bool queries`.  One of the ways is to add multiple queries under the `should clause`. 

**Adding multiple queries under the should clause**

This approach ensures that you maintain a high recall but also offers a way to present more precise search results at the top of your search results.

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        }
      ],
      "should": [
        {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        },
        {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        },
        {
          "Enter match or match_phrase here": {
            "Enter the name of the field": "Enter the value you are looking for"
          }
        }
      ]
    }
  }
}
```
Let's say you want to run a search for news headlines with the phrase "Michelle Obama" in the field headline. But you want to favor articles that mention her biography "Becoming", and terms like "women" and "empower".

To do this, you can add multiple `queries` to the `should clause`.

This will cast a wider net because none of the `queries` in the should clause need to match. However, the ones that match the `queries` under the `should clause` will be given a higher score and placed higher in the search results.

This approach allows you to maintain a high recall but also gives you a way to customize the precision of top hits.

Example:
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
      "should": [
        {
          "match": {
            "headline": "Becoming"
          }
        },
        {
          "match": {
            "headline": "women"
          }
        },
        {
          "match": {
            "headline": "empower"
          }
        }
      ]
    }
  }
}
```
Expected response from Elasticsearch:

Adding many `queries` under the `should clause` did not reduce the number of hits(207). However, it favored documents that match the `queries` in the `should clause` and improved the precision of top search results. 

![image](https://user-images.githubusercontent.com/60980933/108548611-51a8f980-72a9-11eb-8310-0fe14286e437.png)

