---
description: is responsible for editing record in a given resource
---

# Edit

**Endpoint:** `/api/resources/[RESOURCE-ID]/records/[RECORD-ID]/edit`

**Method:** POST

**Request payload:**&#x20;

* `FormData` Object with all required fields for the given resource

**Response:**

* `redirectUrl` - URL that the user should be directed to after a successful update
* `notice`&#x20;
  * `message` - the message that is later displayed in the dashboard
  * `type` - a type of response, possible values are `success`,`error`,`info`
* `record` - record you're requesting
  * `params` - all record data&#x20;
  * `id` - record id
  * `title` - record title
  * `recordActions`- list all actions and their parameters available on this record
  * `bulkActions`- list of all bulk actions and their parameters available on this record
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

Response:

```json
{
   "redirectUrl":"/admin/resources/User",
   "notice":{
      "message":"Successfully updated given record",
      "type":"success"
   },
   "record":{
      "params":{
         "_id":"63d3b2c982bf27f5606e44eb",
         "firstName":"Admin Name",
         "lastName":"Admin Surname",
         "gender":"male",
         "email":"admin1@adminjs.com",
         "isMyFavourite":true,
         "__v":0
      },
      "populated":{
         
      },
      "baseError":null,
      "errors":{
         
      },
      "id":"63d3b2c982bf27f5606e44eb",
      "title":"admin1@adminjs.com",
      "recordActions":[
         {
            "name":"show",
            "actionType":"record",
            "icon":"Screen",
            "label":"Show",
            "resourceId":"User",
            "guard":"",
            "showFilter":false,
            "showResourceActions":true,
            "showInDrawer":false,
            "hideActionHeader":false,
            "containerWidth":1,
            "layout":null,
            "variant":"default",
            "parent":null,
            "hasHandler":true,
            "custom":{
               
            }
         },
         {
            "name":"edit",
            "actionType":"record",
            "icon":"Edit",
            "label":"Edit",
            "resourceId":"User",
            "guard":"",
            "showFilter":false,
            "showResourceActions":true,
            "showInDrawer":false,
            "hideActionHeader":false,
            "containerWidth":1,
            "layout":null,
            "variant":"default",
            "parent":null,
            "hasHandler":true,
            "custom":{
               
            }
         },
         {
            "name":"delete",
            "actionType":"record",
            "icon":"TrashCan",
            "label":"Delete",
            "resourceId":"User",
            "guard":"Do you really want to remove this item?",
            "showFilter":false,
            "showResourceActions":true,
            "component":false,
            "showInDrawer":false,
            "hideActionHeader":false,
            "containerWidth":1,
            "layout":null,
            "variant":"danger",
            "parent":null,
            "hasHandler":true,
            "custom":{
               
            }
         }
      ],
      "bulkActions":[
         {
            "name":"bulkDelete",
            "actionType":"bulk",
            "icon":"Delete",
            "label":"Delete all",
            "resourceId":"User",
            "guard":"",
            "showFilter":false,
            "showResourceActions":true,
            "showInDrawer":true,
            "hideActionHeader":false,
            "containerWidth":"500px",
            "layout":null,
            "variant":"danger",
            "parent":null,
            "hasHandler":true,
            "custom":{
               
            }
         }
      ]
   }
}
```
