---
description: allows you to list and filer all the records for a given resource
---

# List

**Endpoint:** `/api/resources/[RESOURCE-ID]/actions/list`

**Method:** GET

**Request params:**&#x20;

* `direction` - sorting direction, possible values `asc`,`desc`
* `sortBy` - name of the sorting column&#x20;
* `page` - requested page number&#x20;
* `perPage` - number of records per page (max `500)`
* `filers.[field_name]` - filters applied&#x20;

**Response:**

* `meta`
  * `total` - total number of records in the resource
  * `perPage` - number of records in a single page
  * `page` - number of requested page
  * `direction`- sorting direction, possible values `asc`,`desc`
  * `sortBy`- id of the sorting column
* `records` - list of records with resource metadata

**Example**&#x20;

[_https://demo.adminjs.com_/admin/api/resources/Admin/actions/list?direction=desc\&sortBy=\_id\&filters.email=admin\&page=1](https://demo.adminjs.com/admin/api/resources/Admin/actions/list?direction=desc\&sortBy=\_id\&filters.email=admin\&page=1)

```json
{
   "meta":{
      "total":1,
      "perPage":10,
      "page":1,
      "direction":"desc",
      "sortBy":"_id"
   },
   "records":[
      {
         "params":{
            "_id":"62d50386c2d13cd087a10e3a",
            "email":"admin@example.com",
            "password":"$argon2id$v=19$m=4096,t=3,p=1$PFUAZpgSO1XwfnksafaV2Q$+vJ1hrmDAY70Us5iz5bNttDRCOAxLGIAOFaol0KrcjI",
            "__v":0
         },
         "populated":{
            
         },
         "baseError":null,
         "errors":{
            
         },
         "id":"62d50386c2d13cd087a10e3a",
         "title":"admin@example.com",
         "recordActions":[
            {
               "name":"show",
               "actionType":"record",
               "icon":"Screen",
               "label":"Show",
               "resourceId":"Admin",
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
            }
         ],
         "bulkActions":[
            
         ]
      }
   ]
}
```
