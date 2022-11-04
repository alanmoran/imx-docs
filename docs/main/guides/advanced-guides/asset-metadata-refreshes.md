---
title: 'Refresh asset metadata'
slug: '/asset-metadata-refreshes'
sidebar_position: 2
keywords: [imx-games]
---

Metadata refreshes update off-chain asset metadata.

[Read more about asset metadata.](/docs/asset-metadata)

This [page](https://docs.x.immutable.com/sdk-docs/core-sdk-ts/metadata-refresh/) contains code samples that explain how to use the metadata refresh API.

## Use cases

Metadata refreshes help games:

- Support game play mechanics that rely on metadata such as levelling up characters, crafting and merging
- Refine the game balance
- Refine and correct game content such art and copy
- Delay reveals of primary sales
- Improve search discoverability through metadata

## Requirements

In order for a refresh to be successful, first:

- Create and deploy the [Metadata API](/docs/minting-on-immutable-x#metadata-api), as it's the source of truth for asset metadata
- For reveals, update your Metadata API only once the asset is ready to be revealed. The flow for reveals would be something like:
  - Mint an asset
  - Once ready to reveal, update the Metadata API with the properties to be revealed
  - Refresh metadata for the assets to be revealed
- Ensure your Metadata API availability is aligned with the requirements. The metadata refresh service will concurrently request metadata for each asset from your Metadata API and requires a response time of less than 3 seconds per request in order to successfully update the metadata for that asset. If the request to the Metadata API fails it will be retried once more before that particular asset is [marked as failed.](#viewing-metadata-refresh-errors)

## API

Endpoints related to metadata refreshes are denoted via the url resource `/metadata-refreshes`.

The expected maximum time (estimate) to complete a refresh is approximately 48 hours. Updated metadata values will only be reflected in downstream systems e.g. marketplaces once the refresh for that asset is complete.

### Authentication

Metadata refresh endpoints are protected via authentication - only the project owner will be able to call these endpoints.

The following headers are required for project owner authentication:

- `x-imx-eth-address` - the Ethereum address of the project owner making the request
- `x-imx-eth-signature` - the signature generated by the project owner
- `x-imx-eth-timestamp` - the timestamp used to generate the signature

[View how to generate project owner authentication headers.](/docs/generate-imx-signature/)

### Requesting a metadata refresh

A metadata refresh can be requested by calling:

```json
POST /v1/metadata-refreshes
```

and specifying the collection address and IDs of the tokens requiring a refresh:

```json
{
  "collection_address": "0x9a48b1b27743d807331d06ecf0bfb15c06fdb58d", // required
  "token_ids": ["1", "2"] // required
}
```

The user will receive the following response with a HTTP status code of **202 Accepted**:

```json
{
  "refresh_id": "21566bcb-4263-48bb-a495-602ba9a12f2c"
}
```

The `refresh_id` acts as a receipt and confirms that the refresh has been initiated and will be processed. It can be used to query the status of a particular refresh.

Projects can programmatically call the refresh endpoint at a specific time or based on a specific event in their system. There is currently no way to cancel a refresh once requested.

[View the OpenAPI specification for requesting a metadata refresh.](/reference#/operations/Request%20a%20metadata%20refresh)

### Viewing the status of a metadata refresh

The status of a refresh can be queried by using the `refresh_id` with the following endpoint:

```json
GET /v1/metadata-refreshes/:refresh_id
```

which will return the following response:

```json
{
  "refresh_id": "21566bcb-4263-48bb-a495-602ba9a12f2c",
  "status": "completed",
  "collection_address": "0x9a48b1b27743d807331d06ecf0bfb15c06fdb58d",
  "started_at": "2022-08-31T01:59:59.806449Z",
  "completed_at": "2022-08-31T02:00:09.786143Z",
  "summary": {
    "succeeded": 2,
    "failed": 0,
    "pending": 0
  }
}
```

#### Status

The status field of a refresh can be the following values:

- `queued` - the refresh has been requested but no tokens have been processed
- `in_progress` - the refresh has started processing and at least 1 token has been processed
- `completed` - all tokens have been processed

#### Summary

The summary field of a refresh displays the counts of the status of individual tokens belonging to that refresh:

- `succeeded` - the metadata for this token has been successfully updated
- `failed` - the metadata for this token was not able to be updated
- `pending` - the token has not been processed

Another endpoint is available to [investigate the reason why the metadata for a particular token could not be updated.](#viewing-metadata-refresh-errors)

[View the OpenAPI specification for metadata refresh status.](/reference#/operations/Get%20metadata%20refresh%20results)

### Viewing metadata refresh errors

If 2 non-successful attemps have been made to retrieve the updated metadata values from the [Metadata API](/docs/minting-on-immutable-x#metadata-api), that token will be marked as `failed`. A more detailed explanation of why token metadata could not be updated for a particular refresh can be queried with the following endpoint:

```json
GET /v1/metadata-refreshes/:refresh_id/errors
```

which will return a paginated list of tokens within that refresh that could not be updated and their respective error details:

```json
{
  "result": [
    {
      "token_id": "1",
      "collection_address": "0x9a48b1b27743d807331d06ecf0bfb15c06fdb58d",
      "client_token_metadata_url": "https://metadata-api.io/1",
      "client_response_status_code": 404,
      "client_response_body": "404 page not found\n",
      "error_code": "invalid_response_status_code",
      "created_at": "2022-09-06T02:14:04.935946Z"
    }
  ],
  "cursor": "eyJjcmVhdGVkX2F0IjoiMjAyMi0wOS0wNlQwMjoxNDowNC45MzU5NDZaIiwiaWQiOiIyYzU2ZDE5Ni1mY2ViLTQ1MzItYmMzOC0zZmZlMTFlOTM4Y2UifQ",
  "remaining": 0
}
```

Each item in the results array represents one failed token for that particular refresh.

- `client_token_metadata_url` - the url which was used to perform the Metadata API request
- `client_response_status_code` - the response status code that was received from the Metadata API
- `client_response_body` - any error details that were provided as a response body by the Metadata API
- `error_code` - describes why the metadata could not be updated

#### Error codes

<table>
  <thead>
  <tr>
    <th>
      Error code
    </th>
    <th>
      Description
    </th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>
      <code>unable_to_retrieve_metadata</code>
    </td>
    <td>
      The metadata refresh service was unable to get a response from the Metadata API, for example the API timed out
    </td>
  </tr>
  <tr>
    <td>
      <code>invalid_response_status_code</code>
    </td>
    <td>
      The metadata refresh service received a non 200 response code when making a request to the Metadata API
    </td>
  </tr>
  <tr>
    <td>
      <code>unable_to_decode_response</code>
    </td>
    <td>
      The response received by the Metadata API was unable to be decoded
    </td>
  </tr>
  <tr>
    <td>
      <code>token_not_found</code>
    </td>
    <td>
      The token does not exist in our system
    </td>
  </tr>
  </tbody>
</table>

[View the OpenAPI specification for metadata refresh errors.](/reference#/operations/Get%20metadata%20refresh%20errors)

### Getting a list of all metadata refreshes

Users can fetch a list of all metadata refreshes in order to find a particular `refresh_id`, which can then be used to query the status of a particular refresh:

```json
GET /v1/metadata-refreshes
```

The above endpoint will return a list of refreshes ordered by `created_at` , showing the most recent refreshes first.

##### Filter by collection address

The endpoint results can also be filtered by using the the query parameter `collection_address`:

```json
GET /v1/metadata-refreshes?collection_address=COLLECTION_ADDRESS
```

The response returns a paginated list of refreshes:

```json
{
  "result": [
    {
      "refresh_id": "ff52e750-2cc6-4455-b1ae-95a4280e4826",
      "status": "completed",
      "collection_address": "0x9a48b1b27743d807331d06ecf0bfb15c06fdb58d",
      "started_at": "2022-09-06T02:11:03.065286Z",
      "completed_at": "2022-09-06T02:14:04.945285Z"
    }
  ],
  "cursor": "eyJjcmVhdGVkX2F0IjoiMjAyMi0wOC0zMFQwNzozODoxMS4xMzczMTRaIiwiaWQiOiI2YzhlMDNjNS04ZWYxLTQzNmQtOWNjZS1iM2ExNDdmYzc3NGIifQ",
  "remaining": 0
}
```

[View the OpenAPI specification for a list of metadata refreshes.](/reference#/operations/Get%20a%20list%20of%20metadata%20refreshes)

## Metadata refresh limits

There are limits on the number of the number of metadata refresh requests made per project, per hour.

- For a particular project, **5** metadata refresh requests can be made per hour
- Up to **1000** tokens can be requested per refresh

Whenever a [refresh is requested](#requesting-a-metadata-refresh) or when a refresh limit has been reached, the following headers will be returned:

- `x-imx-refreshes-limit` - the amount of refreshes that can be requested per hour
- `x-imx-remaining-refreshes` - the amount of refreshes remaining for the current hour interval
- `x-imx-refresh-limit-reset` - a UNIX timestamp indicating when the limit resets by rolling over into the next hour interval

If the refresh limit has been reached, the user will receive a response with a HTTP status code of **429 Too Many Requests**:

```json
{
  "code": "refresh_limit_exceeded",
  "message": "Refresh limit exceeded"
}
```

:::info Additional requests
If you need to perform additional refresh requests, you can [contact support](https://support.immutable.com).
:::