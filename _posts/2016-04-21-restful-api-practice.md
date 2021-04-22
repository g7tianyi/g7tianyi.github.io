---
layout: post
title: REST-API Practices
categories: tech
tags:
  - REST-ful
  - API
comments: true
featured: false
published: true
---

This post notes down the REST-API practices per my working experiences and understanding. I don't think it's a standard or guideline, read it if you're leisure enough and read it with a skeptical mind.

## 1 URI

URI represents resources, which in general, corresponds to entity defined in Database.

### 1.1 URI Spec

- Encode your query string
- Noun`s` in URI standards for a set of resources

### 1.2 Resource VS Resources

#### 1.2.1 Resources

```bash
/libraries # All libraries
/library/1/books # All books of library 1
/libraries?in_ids=1,2,3 # Libraries whose ID is 1, 2 or 3
```

#### 1.2.2 Resource

```bash
/library/1 # Library whose ID is 1
```

### 1.3 Try not to make your URI too long

`/` denotes the layer/hierarchy in URI. In REST-design, the hierarchy can be viewed as a `navigation coordinate`, the value of each `coordinate` is the entity ID. Yet with the hierarchy of the URI goes deeper, more difficulties come along, personally, `GET /books?library=1&category=3&subCategory=19` make more sense than `GET /library/1/category/3/subCategory/19/books`.

In general, layers shall not exceed 2, and 4 is the upper limit. 

## 2 HTTP Request

Always use standard `HTTP` method to operate (i.e., CRUD) resource.

| Method | Description | Sample |
|:-------|:------------|:-------|
| **GET** | Query resource, return resource entity. Case 1 is GET by ID, case 2 is GET by filter expression | See below |
| **GET** | Query resources, return list of resources meet the filter expression and pagination information | See below |
| **POST** | Create single entity | `POST /book`, `POST /library/1/book` |
| **POST** | Create multiple entities | `POST /books`, `POST /library/1/books` |
| **PUT** | Update single entity, client must provide all fields of the entity | `PUT /book/1`, `PUT /library/1/book/1` |
| **PUT** | Update multiple entities, client must provide all fields of the entities | `PUT /books`, `PUT /libraries/1/books` |
| **PATCH** | Update single entity, client could only provide the fields needs to be updated | `PATCH /book/1`, `PATCH /library/1/book/1` |
| **PATCH** | Update multiple entities, client could only provide the fields needs to be updated | `PATCH /books`, `PATCH /library/1/book/1` |
| **DELETE** | Delete single entity | `DELETE /book/1`, `DELETE /library/1/book/1` |
| **DELETE** | Delete multiple entities, provide the list of entity ID in RequestBody | `DELETE /library/1/books` |

For `HEAD` and `OPTION`, I found Swagger does a better work, maybe I'll find more in the future. 

### 2.1 Filter Expressions

In general, filter expressions is used in `GET`, without any particular reasons, don't apply filter expressions to  `CUD` APIs.   

| Condition | Sample | Description |
|:----------|:-------|:------------|
| **Equals** | `name=jack&...` | - |
| **GreaterThan** | `gt_age=18&...` | - |
| **LessThan** | `lt_age=60&...` | - |
| **NotEquals** | `ne_gender=1&...` | - |
| **FuzzyMatching** | `like_name=jack&...` | Fuzzy matching only works for `string` field. |
| **Location** | `ids=1,2,3&...` | Locate entity whose ID is 1, 2, 3. Location only applicable to `ID`. |
| **Sort** | `sort=-age,createdAt&...` | `age` in descending order, `createdAt` in ascending order. |
| **Pagination** | `pageStart=100&pageSize=10&...` | Get entities in position 100~110. |
| **Projection** | `projection=id,name,age&...` | Specify fields you care. |

> Combine `lt_` and `gt_` to express `range`.

### 2.2 Tags

Create a tag for frequently-used, complex queries, this is useful to reduce the maintenance burden, and ease the pain of auto-testing.

For example, consider to create an API `GET /library/1/newBooks` for `GET /library/1/books?status=1&sort=-createdAt`. 

### 2.3 Format

- `application/json`. Forget the efficiency, everyboy loves JSON. 
- `multipart/form-data`. Used in file upload case.
- `application/x-www-form-urlencoded`. Basically, I don't use it, 😂.

## 3 HTTP Response

### 3.1 No Extra Wrap

I think it waste unnecessary bandwidth, and the client need extra boring operation to unpack the response payload.

Don't do this,

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "name": "jack",
    "age": 30
  }
}
```

Do this instead,

```json
{
  "id": 1,
  "name": "jack",
  "age": 30
}
```

### 3.2 Don't Expose Unnecessary Details on Error Response

To be specific, don't return internal error logs if the client receive status code 500 and don't know what to do,  personally, the client developer shall then reach backend developer directly. On the other hand, suggestive error messages are OK to send, for example,  

```json
{
  "timestamp": 1524314971425,
  "status": 500,
  "error": "Internal Server Error",
  "exception": "com.netflix.zuul.exception.ZuulException",
  "message": "TIMEOUT"
}
```

This payload indicate heavy load in backend, the backend is deploying a new release or restart, client could consider
low the request frequency. 

### 3.3 Response Body of HTTP Methods

| HTTP method | Response |
|:----------|:------------|
| **GET** | `Single entity`, `entity list`, `entity list + pagination` | 
| **POST** | ID or list of IDs of the newly created entity/entities | 
| **PUT** | _Empty_ |
| **PATCH** | _Empty_ |
| **DELETE** | _Empty_ |

#### 3.3.1 Some Aggrements of JSON

- Use timestamp (milliseconds) to represent time, let the client of parse it accordingly, e.g., 
  [moment.js](http://momentjs.com/)
- Don't flush `null` fields

#### 3.3.2 Pagination

Sample:

```json
{
  "data": [],
  "page": {
    "pageStart": 100,
    "pageSize": 10,
    "total": 589
  }
}
```

## 4 Error Handling

### 4.1 General Rules

- Use standard HTTP status code, don't be a smart ass, don't define your own
- Don't return status code 200 upon error, client may cache successful HTTP request
- Don't expose unnecessary details

### 4.1 Business Error

- In general, return `4XX`
- `4XX` should be set to meaningful value according to the business context, e.g., 404 for not found, etc. 
- Return error message, i.e., `library id can't be null`, if the backend don't want to support i18n, return i18n key is also acceptable

### 4.2 Non-Business Error

- In general, this is caused by internal bugs, e.g., NPE, Database timeout, OOM, etc.
- Return error 500 with detailed logs, don't expose details, only return `Internal Server Error`

### 4.3 Status Code

| Status Code | Description | 
|:------------|:------------|
| **400** | Bad Request, used in paratemters validation. |
| **401** | Unauthorized, Unauthenticated access. If authenticated but still has no permission, return 403, i.e., Unauthorization, used in Gateway. |
| **403** | Forbidden, generally used in Gateway. |
| **404** | Not Found. |
| **429** | Too many requests. Used in request limit control. |
| **410** | Gone. Indicate a deprecated and unavailable API. |
| **301** | Moved Permanently. Indicate a deprecated yet still available API. |
| **500** | Internal Server Error. |
| **503** | Service Unavailable. Thrown by containers like Tomcat or Nginx, don't return in your code. |

## 5 API Version

- Put API version in URI: `GET /v1/user/1`
- Put API version in Header：`X-Api-Version: 1`

Use the former one, we love clearness and simplicity. 

One thing worth mentioning, if you do upgrade the API version, e.g., an API moved to a new URI, return `301` or `410` according to your business requirement, at the very least, don't return `400` or `500`.

## 6 Security

### 6.1 Channel Security

HTTP itself has no any mechanism in regard to security, so basically we must setup HTTPS to encrypt data between the server and the client.

To some extent, I don't know the wording `channel security` is the correct terminology, 😂. 

### 6.2 Data Integrity

The problem of data integrity comes by when we trying to update data, to be specific, we need to make sure the data the request want to modify is the same the server maintains. `ETag` in HTTP header is the commonly used to make it.

ETag is the unique version number of the resource/entity, when request the resource, RESTful API should return the resource/entity as well as the ETag, in the meanwhile, client should provide `If-Match` in the HTTP header to update resource/entity, CAS principle may occur to you~ But if data integrity really means something, rigorous parallel control is the thing you shall really pay attention.

### 6.3 API Request Control

- **Whitelist of URL path, Query string, etc.** Sometimes, we may need to reserve some parameters, in this case,  return 403 directly.
- **Access token.** Backend server will return a access token to the client and as the client to provide the token to request data, moreover, we could ask the client to refresh the token within a predefined time interval. If the token is empty, incorrect or expired, return 403 directly.
- **[OAuth](https://en.wikipedia.org/wiki/OAuth)**. The full description of OAuth is quite tedious. Here we consider OAuth a better way to get and refresh the access token (thus the OAuth procedure is simplified). Upon login, the  server return a `accessToken` and a `refreshToken` to the client, as the name indicates, the client should use the  `refreshToken` to get a new `accessToken` (maybe a `refreshToken` as well), the expiration time is longer than the   `accessToken`, e.g., `accessToken` expires in 15 minutes, while `refreshToken` could live 30 days.
- **[HMAC](https://en.wikipedia.org/wiki/HMAC).** This is the upgrade version of access token, for each request, the client must provide a request signature in the request header. For example,  `signature = MD5(method, path, timestamp, accessToken)`, method is `GET`, `PUT`, etc. Backend server will calculate  the signature in the same algorithm and return 403 if unmatched. 

> Note, 
> 
> - More complex the request control policy is, more cost you'll have to pay, make your choice.
> - Generally, apply the request control policy in Gateway or Request Filter (tomcat), if you're using Spring Cloud, you can do it in Zuul as well, as the Jigsaw declares, `The choice is yours`.

### 6.4 API Request Limit

You are right, this is to avoid DDOS attack, besides, it could also keep us from id/directory iteration attack.

Firstly, we could apply different limit policy for different resources and roles (VIP or Paid users).

Secondly, The server should report the limit statistics to the client,

- **X-RateLimit-Limit**, indicates the maximum requests the client could fire every hour.   
- **X-RateLimit-Available**, indicates the available requests the client could fire in the current time window.
- **X-RateLimit-Remaining**, indicates the remaining time to reset `X-RateLimit-Rest`.

Last but not least, status code `429` (Too many requests) is preferred if the client do exceed the request limit.

## 7 Test and Auto Test

> To be added.
