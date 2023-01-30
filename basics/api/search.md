---
description: >-
  allows you to search records in a given resource by a query string (by default
  it's the title property)
---

# Search

**Endpoint:** `/api/resources/[RESOURCE-ID]/actions/search/[SEARCH-PHRASE]?[SEARCH-CONDITIONS]`

**Method:** GET

**Request params:**&#x20;

* `title` - searching by title
* `filers.[field_name]` - searching by field values
* `page` - requested page number&#x20;
* `perPage` - number of records per page (max `500)`
* `sortBy` - id of the sorting column&#x20;
* `direction` - sorting direction, possible values `asc`,`desc`

**Response:**

* `records` - list of records with resource metadata
  * `record` - record you're requesting
    * `params` - all record data&#x20;
    * `id` - record id
    * `title` - record title
    * `recordActions`- list all actions and their parameters available on this record
    * `bulkActions`- list of all bulk actions and their parameters available on this record

**Example:**

Endpoint: [https://adminjs-demo.herokuapp.com/admin/api/resources/categories/actions/search/Games](https://adminjs-demo.herokuapp.com/admin/api/resources/categories/actions/search/Games)

```json
{
   "records":[
      {
         "params":{
            "id":2,
            "name":"Games",
            "createdAt":"2023-01-30T13:01:39.597Z",
            "updatedAt":"2023-01-30T13:01:39.597Z"
         },
         "populated":{
            
         },
         "baseError":null,
         "errors":{
            
         },
         "id":2,
         "title":"Games",
         "recordActions":[
            {
               "name":"show",
               "actionType":"record",
               "icon":"Screen",
               "label":"Show",
               "resourceId":"categories",
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
               "resourceId":"categories",
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
               "resourceId":"categories",
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
               "resourceId":"categories",
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
   ]
}
```
