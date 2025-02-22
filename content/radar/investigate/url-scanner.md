---
pcx_content_type: reference
title: URL Scanner (beta)
weight: 7
---

{{<heading-pill style="beta">}} Radar's URL Scanner {{</heading-pill>}}

In order to better understand Internet usage around the world, Radar provides a URL Scanner at [https://radar.cloudflare.com/scan](https://radar.cloudflare.com/scan).

To make your first URL scan using the API, you must obtain a URL Scanner specific [API token](/fundamentals/api/get-started/create-token/). Create a Custom Token with _Account_ > _URL Scanner_ in the **Permissions** group, and select _Edit_ as the access level.

Once you have the token, and you know your `accountId`, you are ready to make your first request to the API at `https://api.cloudflare.com/client/v4/accounts/<accountId>/urlscanner/`.

## Submit URL to scan

In order to submit a URL to scan, the only required information is the URL to be scanned in the `POST` request body:


```bash
curl --request POST \
	--url https://api.cloudflare.com/client/v4/accounts/<accountId>/urlscanner/scan \
	--header 'Content-Type: application/json' \
    --header "Authorization: Bearer <API_TOKEN>" \
	--data '{
		"url": "https://www.example.com",
	}'
```

By default, the report will have a `Public` visibility level, which means it will appear in the [recent scans](https://radar.cloudflare.com/scan#recent-scans) list and in search results. It will also include a single screenshot with desktop resolution.

A successful response will have a status code of `200` and be similar to the following:

```json
{
  "errors": [],
  "messages": [{
      "message": "Submission successful"
  }],
  "result": {
    "time": "2022-09-15T00:00:00Z",
    "url": "https://www.example.com",
    "uuid": "095be615-a8ad-4c33-8e9c-c7612fbf6c9f",
    "visibility": "Public"
  },
  "success": true
}
```

The `result.uuid` property in the response above identifies the scan and will be required when fetching the scan report.

### Submit a custom URL Scan

Here's an example request body with some custom configuration options:

```json
{
	"url": "https://example.com",
	"screenshotsResolutions": [
		"desktop", "mobile", "tablet"
	],
	"customHeaders": {
		"user-agent": "My-custom-user-agent",
	},
	"visibility": "Unlisted"
}
```

Above, the visibility level is set as `Unlisted`, which means that the scan report won't be included in the [recent scans](https://radar.cloudflare.com/scan#recent-scans) list nor in search results. In  effect, only users with knowledge of the scan ID will be able to access it.

There will also be three screenshots taken of the webpage, one per target device type. The [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) HTTP Header will be set as "My-custom-user-agent". Note that you can set any custom HTTP header, including [Authorization](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization).

## Get scan report

Once the URL Scan submission is made, the current progress can be checked by calling `https://api.cloudflare.com/client/v4/accounts/<accountId>/urlscanner/scan/<scanId>`. The `scanId` will be the `result.uuid` value returned in the previous response.

While the scan is in progress, the HTTP status code will be `202`, once it's finished it will be `200`. Clients are advised to poll every 10-30 seconds.

The response will include, among others, the following top properties in `result.scan`:

- `task` - Information on the scan submission.
- `page` - Information pertaining to the primary request (for example, response cookies) and the webpage itself (e.g. console messages).
- `meta` - Meta processors output including detected technologies, categories, rank and others.
- `ips` - IPs contacted.
- `asns` - AS Numbers contacted.
- `geo` - GeoIP information derived from contacted IPs.
- `domains` - Hostnames contacted, including `dns` record information.
- `links` - Outgoing links detected in the DOM.
- `performance` - Timings as given by the [`PerformanceNavigationTiming`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming) interface.
- `certificates` - TLS certificates of HTTP responses.
- `verdicts` - Verdicts on malicious content.

Some examples of more specific properties include:

- `task.uuid` - ID of the scan.
- `task.effectiveUrl` - URL of the primary request, after all HTTP redirects.
- `task.success` - Whether scan was successful or not. Scans can fail for various reasons, including DNS errors.
- `task.status` - Current scan status, for example, `Queued`, `InProgress`, or `Finished`.
- `meta.processors.categories` - Cloudflare categories of the main hostname contacted.
- `meta.processors.securityRiskCategories` - Cloudflare categories, representing a security risk, of the main hostname contacted.
- `meta.processors.phishing` - What kind of phishing, if any, was detected.
- `meta.processors.rank` - [Cloudflare Radar Rank](http://blog.cloudflare.com/radar-domain-rankings/) of the main hostname contacted.
- `meta.processors.tech` - What kind of technologies were detected as being in use by the website, with the help of [Wappalyzer](https://github.com/wappalyzer/wappalyzer).
- `page.country` - GeoIP country name of the main IP address contacted.
- `page.cookies` - Cookies set by the page.
- `page.console` - JavaScript console messages
- `page.js.variables` - Non-standard JavaScript global variables.
- `page.securityViolations` - {{<glossary-tooltip term_id="content security policy (CSP)" link="https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP">}}CSP{{</glossary-tooltip>}} or [SRI](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) violations.
- `verdicts.overall.malicious` - Whether the website was considered malicious _at the time of the scan_. Please check the remaining properties for each subsystem(s) for specific threats detected.

The [Get URL Scan](/api/operations/urlscanner-get-scan) API endpoint documentation contains the full response schema.

In order to fetch the scan's [screenshots](/api/operations/urlscanner-get-scan-screenshot) or full [network log](/api/operations/urlscanner-get-scan-har), please visit the corresponding endpoints' documentation.


## Search scans

`Public` scans can also be searched for. In order to search for scans to the hostname `google.com`, use the query parameter `page_hostname=google.com`:

```
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<accountId>/urlscanner/scan?page_hostname=google.com \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer <API_TOKEN>"
```

Search results will also include your _own_ `Unlisted` scans.

If, instead, you wanted to search for scans which made at least one request to the hostname `cdnjs.cloudflare.com` - e.g. sites that use a JavaScript library hosted at `cdnjs.cloudflare.com`  - use the query parameter `hostname=cdnjs.cloudflare.com`:

```
curl --request GET \
  --url https://api.cloudflare.com/client/v4/accounts/<accountId>/urlscanner/scan?hostname=cdnjs.cloudflare.com \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer <API_TOKEN>"
```

Check `https://developers.cloudflare.com/api/operations/urlscanner-search-scans` for the full list of available options.