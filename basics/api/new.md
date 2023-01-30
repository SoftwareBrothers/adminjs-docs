---
description: is responsible for creating a new record in a given resource
---

# New

**Endpoint:** `/api/resources/[RESOURCE-ID]/actions/new`

**Method:** POST

**Request payload:**&#x20;

* `FormData` object with all required fields for the given resource

**Response:**

* `meta`
  * `total` - total number of records in the resource
  * `perPage` - number of records in a single page
  * `page` - number of requested page
  * `direction`- sorting direction, possible values `asc`,`desc`
  * `sortBy`- id of the sorting column
* `records` - list of records with resource metadata

**Example**&#x20;

Endpoint: [https://demo.adminjs.co/admin/api/resources/User/actions/new](https://adminjs-demo.herokuapp.com/admin/api/resources/User/actions/new)

Payload:&#x20;

<pre class="language-json"><code class="lang-json">{
    email: "client@adminjs.co",
<strong>    firstName: "ClientName",
</strong><strong>    lastName: "ClientSurname",
</strong><strong>    gender: "male",
</strong><strong>    isMyFavourite: true
</strong>}
</code></pre>
