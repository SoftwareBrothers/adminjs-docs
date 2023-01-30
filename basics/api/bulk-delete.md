---
description: is responsible for deleting multiple records
---

# Bulk Delete

**Endpoint:** `/api/resources/[RESOURCE-ID]/bulk/bulkDelete`

**Method:** POST

**Request:**

* `recordIds` - list of ids of the records to be deleted&#x20;

**Response:**

* `records` - list of records with resource metadata
* `record` - record you're requesting
  * `params` - all record data&#x20;
  * `id` - record id
  * `title` - record title
  * `recordActions`- list all actions and their parameters available on this record
  * `bulkActions`- list of all bulk actions and their parameters available on this record
* `notice`&#x20;
  * `message` - the message that is later displayed in the dashboard
  * `type` - a type of response, possible values are `success`,`danger`,`info`
* `redirectUrl` - URL that the user should be directed to after successfully edit

**Example**&#x20;

Endpoint: [https://demo.adminjs.co/admin/api/resources/User/bulk/bulkDelete?recordIds=63d3af2ab1b453f9303c81d0\&recordIds=63d3af2ab1b453f9303c81d1](https://demo.adminjs.co/admin/api/resources/User/bulk/bulkDelete?recordIds=63d3af2ab1b453f9303c81d0\&recordIds=63d3af2ab1b453f9303c81d1)

Response:&#x20;

```json
{
   "records":[
      {
         "params":{
            "_id":"63d3af2ab1b453f9303c81d0",
            "firstName":"Warren",
            "lastName":"Renner",
            "gender":"female",
            "email":"Naomi90@gmail.com",
            "isMyFavourite":true,
            "__v":0
         },
         "populated":{
            
         },
         "baseError":null,
         "errors":{
            
         },
         "id":"63d3af2ab1b453f9303c81d0",
         "title":"Naomi90@gmail.com",
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
      },
      {
         "params":{
            "_id":"63d3af2ab1b453f9303c81d1",
            "firstName":"Jovanny",
            "lastName":"Moore",
            "gender":"male",
            "email":"Vanessa53@hotmail.com",
            "isMyFavourite":false,
            "__v":0
         },
         "populated":{
            
         },
         "baseError":null,
         "errors":{
            
         },
         "id":"63d3af2ab1b453f9303c81d1",
         "title":"Vanessa53@hotmail.com",
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
   ],
   "notice":{
      "message":"successfully removed 2 record",
      "type":"success"
   },
   "redirectUrl":"/admin/resources/User"
}
```
