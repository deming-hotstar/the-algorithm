# Twitter's Recommendation Algorithm

## 1.What components are included in the repo

- data
- model
- framework

| Type | Component | Description                                                                                                                                           |
|------------|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| Data | [tweetypie](tweetypie/server/README.md) | 1 Tweet service: the reading and writing of Tweet data.                                                                                               |
|      | [unified-user-actions](unified_user_actions/README.md) | 2 Real-time stream of user actions on Twitter. (generate and transform client and server-side event)                                                  |
|      | [user-signal-service](user-signal-service/README.md) | 3 Centralized platform to retrieve explicit (e.g. likes, replies) and implicit (e.g. profile visits, tweet clicks) user signals. (like feature-store) |
| Model | [SimClusters](src/scala/com/twitter/simclusters_v2/README.md) | 1 (For Feature: sparse embedding) Community detection and sparse embeddings into those communities.                                                   |
|       | [TwHIN](https://github.com/twitter/the-algorithm-ml/blob/main/projects/twhin/README.md) | 2 (For Feature: dense embedding) Dense knowledge graph embeddings for Users and Tweets.                                                               |
|       | [trust-and-safety-models](trust_and_safety_models/README.md) | 3 Models for detecting NSFW (not safe for work) or abusive content.                                                                                   |
|       | [real-graph](src/scala/com/twitter/interaction_graph/README.md) | 4 Model to predict the likelihood of a Twitter (User interacting with another User).                                                                  |
|       | [tweepcred](src/scala/com/twitter/graph/batch/job/tweepcred/README) | 5 Page-Rank algorithm for calculating Twitter (User reputation).                                                                                      |
|       | [recos-injector](recos-injector/README.md) | 6 Streaming event processor for building (input streams for [GraphJet](https://github.com/twitter/GraphJet) based services).                          |
|       | [graph-feature-service](graph-feature-service/README.md) | 7 Serves graph features for a directed pair of Users (e.g. how many of User A's following liked Tweets from User B).                                  |
|       | [topic-social-proof](topic-social-proof/README.md) | 8 Identifies topics related to individual Tweets.                                                                                                     |
|       | [representation-scorer](representation-scorer/README.md) | 9 Compute scores between pairs of entities (Users, Tweets, etc.) using embedding similarity.                                                          |
| Software framework | [navi](navi/README.md) | 1 High performance, machine learning (model serving) written in Rust.                                                                                 |
|                    | [product-mixer](product-mixer/README.md) | 2 Software framework for building (feeds of content).                                                                                                 |
|                    | [timelines-aggregation-framework](timelines/data_processing/ml_util/aggregation_framework/README.md) | 3 (?)Framework for generating aggregate features in batch or real time.                                                                               |
|                    | [representation-manager](representation-manager/README.md) | 4 Service to retrieve embeddings (i.e. SimClusers and TwHIN).                                                                                         |
|                    | [twml](twml/README.md) | 5 Legacy machine learning framework built on TensorFlow v1.                                                                                           |

**Today's share is the SimClusters part**

## 2.Product Surface

The product surfaces currently included in this repository are the For You Timeline and Recommended Notifications.

### 2.1 For You Timeline

The diagram below illustrates how major services and jobs interconnect to construct a For You Timeline.

![](docs/system-diagram.png)

The core components of the For You Timeline included in this repository are listed below:

| Type | Component | Description                                                                                                                                                                                                                                                                                                                                                        |
|------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Candidate Source | [search-index](src/java/com/twitter/search/README.md) | 1. Find and rank In-Network Tweets. ~50% of Tweets come from this candidate source.                                                                                                                                                                                                                                                                                |
|                  | [cr-mixer](cr-mixer/README.md) | 2. Coordination layer for fetching Out-of-Network tweet candidates from underlying compute services.                                                                                                                                                                                                                                                               |
|                  | [user-tweet-entity-graph](src/scala/com/twitter/recos/user_tweet_entity_graph/README.md) (UTEG)| 2.1 Maintains an in memory User to Tweet interaction graph, and finds candidates based on traversals of this graph. This is built on the [GraphJet](https://github.com/twitter/GraphJet) framework. (also use SimCluster, TwHIN -> community related) Several other GraphJet based features and candidate sources are located [here](src/scala/com/twitter/recos). |
|                  | [follow-recommendation-service](follow-recommendations-service/README.md) (FRS)| 2.2 Provides Users with recommendations for (accounts to follow), and Tweets from those accounts.                                                                                                                                                                                                                                                                  |
| Ranking | [light-ranker](src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/README.md) | Light Ranker model used by search index (Earlybird) to rank Tweets.                                                                                                                                                                                                                                                                                                |
|         | [heavy-ranker](https://github.com/twitter/the-algorithm-ml/blob/main/projects/home/recap/README.md) | Neural network for ranking candidate tweets. One of the main signals used to select timeline Tweets post candidate sourcing.                                                                                                                                                                                                                                       |
| Tweet mixing & filtering | [home-mixer](home-mixer/README.md) | Main service used to construct and serve the Home Timeline. Built on [product-mixer](product-mixer/README.md).                                                                                                                                                                                                                                                     |
|                          | [visibility-filters](visibilitylib/README.md) | Responsible for filtering Twitter content to support legal compliance, improve product quality, increase user trust, protect revenue through the use of hard-filtering, visible product treatments, and coarse-grained downranking.                                                                                                                                |
|                          | [timelineranker](timelineranker/README.md) | Legacy service which provides relevance-scored tweets from the Earlybird Search Index and UTEG service.                                                                                                                                                                                                                                                            |

### 2.2 Recommended Notifications 

The core components of Recommended Notifications included in this repository are listed below:

| Type | Component | Description |
|------------|------------|------------|
| Service | [pushservice](pushservice/README.md) | Main recommendation service at Twitter used to surface recommendations to our users via notifications.
| Ranking | [pushservice-light-ranker](pushservice/src/main/python/models/light_ranking/README.md) | Light Ranker model used by pushservice to rank Tweets. Bridges candidate generation and heavy ranking by pre-selecting highly-relevant candidates from the initial huge candidate pool. |
|         | [pushservice-heavy-ranker](pushservice/src/main/python/models/heavy_ranking/README.md) | Multi-task learning model to predict the probabilities that the target users will open and engage with the sent notifications. |
