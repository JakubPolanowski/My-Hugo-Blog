---
title: "Scraping Realtor Listings"
description: "A brief guide on using Realtor's public albeit undocumented API to pull listings data for Real Estate properties."
date: 2023-03-03T02:27:00-05:00
categories:
    - Programming
    - Web Scraping
tags:
    - Python
    - Requests
    - GraphQL
    - Real Estate
draft: true
---

## Introduction

[Realtor.com](https://www.realtor.com/) uses a GraphQL API. Although not publicly documented the API can be leveraged to pull all the publicly available listings data. 

See **[Scrape Any Website]({{< ref "/post/web-scrape-anything" >}})** for how public facing APIs like this can be discovered.

*Note: This guide will be using Python, although any language capable of sending HTTP requests should be capable of replicating this.*

## The Request

There are four components needs to create a successful request, the URL, headers, the GraphQL query, and finally any parameters for the query.

The URL is simple: `https://www.realtor.com/api/v1/hulk`

And so are the parameters: `{"client_id":"rdc-x","schema":"vesta"}`

### Headers

Headers are crucial, most website will outright reject requests if the certain headers are missing. The reason for this is because these public facing APIs try to dictate that only modern web browsers can be used to fetch the data. Therefore to fetch the data from these APIs, the request must identify itself as a common web browser, as some other miscellaneous values. 

Below are the general purpose headers for use with Realtor.com

```python
headers = {
    "authority": "www.realtor.com",
    "accept": "application/json",
    "accept-language": "en-US,en;q=0.6",
    "content-type": "application/json",
    "origin": "https://www.realtor.com",
    "sec-fetch-dest": "empty",
    "sec-fetch-mode": "cors",
    "sec-fetch-site": "same-origin",
    "sec-gpc": "1",
    "user-agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
}
```

### GraphQL Query

This is the meat of the request. The API is not publicly documented so tweaking the query is a game of trial and error. That being said, the default query essentially captures most of the worthwhile data on a property listing. 

```graphql
query ConsumerSearchMainQuery(
  $query: HomeSearchCriteria!
  $limit: Int
  $offset: Int
  $sort: [SearchAPISort]
  $sort_type: SearchSortType
  $client_data: JSON
  $bucket: SearchAPIBucket
) {
  home_search: home_search(
    query: $query
    sort: $sort
    limit: $limit
    offset: $offset
    sort_type: $sort_type
    client_data: $client_data
    bucket: $bucket
  ) {
    count
    total
    results {
      property_id
      list_price
      primary
      rent_to_own {
        rent
        right_to_purchase
        provider
      }
      primary_photo(https: true) {
        href
      }
      source {
        id
        agents {
          office_name
        }
        type
        spec_id
        plan_id
      }
      community {
        property_id
        permalink
        description {
          name
        }
        advertisers {
          office {
            hours
            phones {
              type
              number
            }
          }
          builder {
            fulfillment_id
          }
        }
      }
      products {
        brand_name
        products
      }
      listing_id
      matterport
      virtual_tours {
        href
        type
      }
      status
      permalink
      price_reduced_amount
      other_listings {
        rdc {
          listing_id
          status
          listing_key
          primary
        }
      }
      description {
        beds
        baths
        baths_full
        baths_half
        baths_1qtr
        baths_3qtr
        garage
        stories
        type
        sub_type
        lot_sqft
        sqft
        year_built
        sold_price
        sold_date
        name
      }
      location {
        street_view_url
        address {
          line
          postal_code
          state
          state_code
          city
          coordinate {
            lat
            lon
          }
        }
        county {
          name
          fips_code
        }
      }
      tax_record {
        public_record_id
      }
      lead_attributes {
        show_contact_an_agent
        opcity_lead_attributes {
          cashback_enabled
          flip_the_market_enabled
        }
        lead_type
        ready_connect_mortgage {
          show_contact_a_lender
          show_veterans_united
        }
      }
      open_houses {
        start_date
        end_date
        description
        methods
        time_zone
        dst
      }
      flags {
        is_coming_soon
        is_pending
        is_foreclosure
        is_contingent
        is_new_construction
        is_new_listing(days: 14)
        is_price_reduced(days: 30)
        is_plan
        is_subdivision
      }
      list_date
      last_update_date
      coming_soon_date
      photos(limit: 2, https: true) {
        href
      }
      tags
      branding {
        type
        photo
        name
      }
    }
  }
}
```

If you're familiar with graphQL, you'll notice that there are some variable included there, below is the default values for those variables.

```python
"variables": {
    "query": {
        "status": ["for_sale", "ready_to_build"],
        "primary": True,
        # use search_location to set location
        "search_location": {"location": "New York, NY"}
    },
    "limit": 42,
    "offset": 0,  # iterate with this
    "sort_type": "relevant",
    "by_prop_type": ["home"]
}
```

In the variables above, there are two particularly import variables:

1. `search_location` used to find properties in a specific location
2. `offset` offset the results by this number
   * For instance if you want your first listing in your response data to be the 43rd result, set `"offset": 42`

**NOTE:** You should avoid changing the `limit` variable. In theory this should allow increasing the number of results that a single query can send back, but in practice increasing it will often result in the API returning an error with no data. 

Beyond that, adjust the query as desired (although be ready to revert changes - it's all trial and error after all).

*Additionally, you can use more advanced search criteria like drawing a polygon search area on the map on realtor.com. Once you submit the query on the website, use chrome dev tools and Insomnia to grab the request payload (see [here]({{< ref "/post/web-scrape-anything" >}})).*

### The Complete Request

```python
import requests

url = "https://www.realtor.com/api/v1/hulk_main_srp"

querystring = {"client_id": "rdc-x", "schema": "vesta"}

payload = {
    "query": """
        query ConsumerSearchMainQuery(
            $query: HomeSearchCriteria!
            $limit: Int
            $offset: Int
            $sort: [SearchAPISort]
            $sort_type: SearchSortType
            $client_data: JSON
            $bucket: SearchAPIBucket
            ) {
            home_search: home_search(
                query: $query
                sort: $sort
                limit: $limit
                offset: $offset
                sort_type: $sort_type
                client_data: $client_data
                bucket: $bucket
            ) {
                count
                total
                results {
                property_id
                list_price
                primary
                rent_to_own {
                    rent
                    right_to_purchase
                    provider
                }
                primary_photo(https: true) {
                    href
                }
                source {
                    id
                    agents {
                    office_name
                    }
                    type
                    spec_id
                    plan_id
                }
                community {
                    property_id
                    permalink
                    description {
                    name
                    }
                    advertisers {
                    office {
                        hours
                        phones {
                        type
                        number
                        }
                    }
                    builder {
                        fulfillment_id
                    }
                    }
                }
                products {
                    brand_name
                    products
                }
                listing_id
                matterport
                virtual_tours {
                    href
                    type
                }
                status
                permalink
                price_reduced_amount
                other_listings {
                    rdc {
                    listing_id
                    status
                    listing_key
                    primary
                    }
                }
                description {
                    beds
                    baths
                    baths_full
                    baths_half
                    baths_1qtr
                    baths_3qtr
                    garage
                    stories
                    type
                    sub_type
                    lot_sqft
                    sqft
                    year_built
                    sold_price
                    sold_date
                    name
                }
                location {
                    street_view_url
                    address {
                    line
                    postal_code
                    state
                    state_code
                    city
                    coordinate {
                        lat
                        lon
                    }
                    }
                    county {
                    name
                    fips_code
                    }
                }
                tax_record {
                    public_record_id
                }
                lead_attributes {
                    show_contact_an_agent
                    opcity_lead_attributes {
                    cashback_enabled
                    flip_the_market_enabled
                    }
                    lead_type
                    ready_connect_mortgage {
                    show_contact_a_lender
                    show_veterans_united
                    }
                }
                open_houses {
                    start_date
                    end_date
                    description
                    methods
                    time_zone
                    dst
                }
                flags {
                    is_coming_soon
                    is_pending
                    is_foreclosure
                    is_contingent
                    is_new_construction
                    is_new_listing(days: 14)
                    is_price_reduced(days: 30)
                    is_plan
                    is_subdivision
                }
                list_date
                last_update_date
                coming_soon_date
                photos(limit: 2, https: true) {
                    href
                }
                tags
                branding {
                    type
                    photo
                    name
                }
                }
            }
        }
    """,
    "variables": {
        "query": {
            "status": ["for_sale", "ready_to_build"],
            "primary": True,
            "search_location": {"location": "New York, NY"}
        },
        "limit": 42,
        "offset": 42,
        "sort_type": "relevant",
        "by_prop_type": ["home"]
    },
    "operationName": "ConsumerSearchMainQuery",
    "callfrom": "SRP",
    "nrQueryType": "MAIN_SRP",
    "isClient": True,
}

headers = {
    "authority": "www.realtor.com",
    "accept": "application/json",
    "accept-language": "en-US,en;q=0.5",
    "content-type": "application/json",
    "origin": "https://www.realtor.com",
    "sec-fetch-dest": "empty",
    "sec-fetch-mode": "cors",
    "sec-fetch-site": "same-origin",
    "sec-gpc": "1",
    "user-agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"
}

response = requests.request(
    "POST", url, json=payload, headers=headers, params=querystring)

print(response.text)
```

## Conclusion

Now you have the full request and can use it to build a loop that gathers all the listings that satisfy your criteria. 

Here is a link to a python notebook in which I've have a demo of this in action.

Beyond that you can work to pull the wildfire risk and flood risks scores from realtor.com (see notebook linked above). Use the official Google Maps API or the public facing Waze API (see this [library](https://github.com/kovacsbalu/WazeRouteCalculator) for ideas), or anything else relating to Real Estate Listings. 

