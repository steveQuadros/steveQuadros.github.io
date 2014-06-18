---
layout: post
title: "Formatting API responses"
date: 2014-06-18 12:14:29 -0400
comments: true
categories: API
---

@TODO - record collections and singular items for each resource to compare


I always like to do a bit of research on best practices before diving into a project.

Event it's something I'm familiar with, it never hurts to get some updated information and see if the experience I've gained since my last effort allows me to interpret information in a new way.

Recently I began building an API for my company to serve as the back end for an angularJS app, iOS (iPad and iPhone), and an android app for various devices.

One of the things that I wanted to investigate was how various APIs handled things like metadata, pagination links, and returning successful responses.

Click one of the below to jump to a particular example:

[Twitter](#twitter)

[Facebook](#facebook)

[Google Plus](#google)

[Github](#github)



<a name="twitter"></a>[Twitter](https://api.twitter.com/1.1/)
---

A successful request to twitter's search API v1.1 will return some meta data regarding pagination and a nested key `results` which is an array of the returned data.


**Pagination**

Twitter asks you to supply a query parameter called "rpp", which is the results per page.

They also have a particularly interesting situation with pagination; because tweets happen so often, requesting tweets between two periods of time would present a huge number of results, so they use `max_id` and `since_id` to limit the number returned.

**Summary**

Twitter uses un-nested meta data included directly in the response.


Example Response (I've truncated the `results`):

{% codeblock lang:javascript  Twitter search results https://dev.twitter.com/docs/api/1/get/search View on twitter api docs site %}
{
  "completed_in":0.031,
  "max_id":122078461840982016,
  "max_id_str":"122078461840982016",
  "next_page":"?page=2&max_id=122078461840982016&q=blue%20angels&rpp=5",
  "page":1,
  "query":"blue+angels",
  "refresh_url":"?since_id=122078461840982016&q=blue%20angels",
  "results":[
    {
      "created_at":"Thu, 06 Oct 2011 19:36:17 +0000",
      "entities":{
        "urls":[
          {
            "url":"http://t.co/L9JXJ2ee",
            "expanded_url":"http://bit.ly/q9fyz9",
            "display_url":"bit.ly/q9fyz9",
            "indices":[
              37,
              57
            ]
          }
        ]
      },
      "from_user":"SFist",
      "from_user_id":14093707,
      "from_user_id_str":"14093707",
      "geo":null,
      "id":122032448266698752,
      "id_str":"122032448266698752",
      "iso_language_code":"en",
      "metadata":{
        "recent_retweets":3,
        "result_type":"popular"
      },
      "profile_image_url":"http://a3.twimg.com/profile_images/51584619/SFist07_normal.jpg",
      "source":"&lt;a href=&quot;http://twitter.com/tweetbutton&quot; rel=&quot;nofollow&quot;&gt;Tweet Button&lt;/a&gt;",
      "text":"Reminder: Blue Angels practice today http://t.co/L9JXJ2ee",
      "to_user_id":null,
      "to_user_id_str":null
    },
    ...
  ],
  "results_per_page":5,
  "since_id":0,
  "since_id_str":"0"
}
{% endcodeblock %}

<a name="facebook"></a>Facebook
---

This is from the [facebook API graphs explorer](https://developers.facebook.com/tools/explorer/) and calls `me/statuses?limit=5&offset=10&fields=id`

**Pagination**

Facebook uses `limit` and `offset` to grab pages.

**Summary**

Facebook also returns a collection (titled "data") and a top level key called "paging". Within the paging element are links to the previous and next set of results.


{% codeblock lang:javascript %}

{
  "data": [
    {
      "id": "THE ID",
      "updated_time": "2014-01-14T14:14:20+0000"
    },
    {
      "id": "THE ID",
      "updated_time": "2014-01-14T03:13:42+0000"
    },
    {
      "id": "THE ID",
      "updated_time": "2014-01-09T01:52:19+0000"
    },
    {
      "id": "THE ID",
      "updated_time": "2013-12-25T06:49:55+0000"
    },
    {
      "id": "THE ID",
      "updated_time": "2013-07-25T01:17:36+0000"
    }
  ],
  "paging": {
    "previous": "https://graph.facebook.com/v2.0/{AnID}/statuses?fields=id&limit=5&since={AnId}&__paging_token={AToken}",
    "next": "https://graph.facebook.com/v2.0/{AnID}/statuses?fields=id&limit=5&until={AnId}&__paging_token={AToken}"
  }
}
{% endcodeblock %}

<a name="google"></a>Google Plus
---

Google plus also uses a JSON only representation of both the data and some meta information. The interesting thing with google is that their represenation for [collections](https://developers.google.com/+/api/latest/people/list) differs from that of [single resources](https://developers.google.com/+/api/latest/people#resource).

**Pagination**

Google plus uses a combination of `maxResults` and a `pageToken`. MaxResults is self-explanitory; pageToken is returned as the JSON field `nextPageToken` and can be added as a query paramter to grab the next set of results. In addition, they also send a `totalItems` field so you know how many items you could potentially be paging through. They do not send full urls to the next, and previous page.

**Summary**

- Collection resources:
    - metadata is sent at the top level
    - the collection of objects is nested inside an array whose key is also at the top level

A collection returns meta data:

{% codeblock lang:javascript %}
{
  "kind": "plus#peopleFeed",
  "etag": etag,
  "selfLink": string,
  "title": string,
  "nextPageToken": string,
  "totalItems": integer,
  "items": [
    ... JSON OBJECTS
  ]
}
{% endcodeblock %}

while a single resource returns a top level object:

{% codeblock lang:javascript %}
{
  "kind": "plus#person",
  "etag": etag,
  "emails": [
    {
      "value": string,
      "type": string
    }
  ],
  "urls": [
    {
      "value": string,
      "type": string,
      "label": string
    }
  ],
  "objectType": string,
  "id": string,
  "displayName": string,
  "name": {
    "formatted": string,
    "familyName": string,
    "givenName": string,
    "middleName": string,
    "honorificPrefix": string,
    "honorificSuffix": string
  },
  ...
  "domain": string
}
{% endcodeblock %}


<a name="github"></a>[Github](https://developer.github.com/v3/users/)
---

**Pagination**

Github uses an HTTP Link header to send the pagination information. They include the next, last, first, and previous links as full urls.

{% codeblock linenos:true %}
Link: <https://api.github.com/search/code?q=addClass+user%3Amozilla&page=3>; rel="next",
<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=34>; rel="last",
<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=1>; rel="first",
<https://api.github.com/search/code?q=addClass+user%3Amozilla&page=1>; rel="prev"
{% endcodeblock %}

<br>
Example Response:

{% codeblock lang:javascript %}
{
  "login": "octocat",
  "id": 1,
  "avatar_url": "https://github.com/images/error/octocat_happy.gif",
  "gravatar_id": "somehexcode",
  "url": "https://api.github.com/users/octocat",
  "html_url": "https://github.com/octocat",
  "followers_url": "https://api.github.com/users/octocat/followers",
  "following_url": "https://api.github.com/users/octocat/following{/other_user}",
  "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
  "organizations_url": "https://api.github.com/users/octocat/orgs",
  "repos_url": "https://api.github.com/users/octocat/repos",
  "events_url": "https://api.github.com/users/octocat/events{/privacy}",
  "received_events_url": "https://api.github.com/users/octocat/received_events",
  "type": "User",
  "site_admin": false,
  "name": "monalisa octocat",
  "company": "GitHub",
  "blog": "https://github.com/blog",
  "location": "San Francisco",
  "email": "octocat@github.com",
  "hireable": false,
  "bio": "There once was...",
  "public_repos": 2,
  "public_gists": 1,
  "followers": 20,
  "following": 0,
  "created_at": "2008-01-14T04:33:35Z",
  "updated_at": "2008-01-14T04:33:35Z"
}
{% endcodeblock %}