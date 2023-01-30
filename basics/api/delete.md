---
description: is responsible for deleting single records
---

# Delete

**Endpoint:** `/api/resources/[RESOURCE-ID]/records/[RECORD-ID]/delete`

**Method:** GET

**Response**

* `record` - record you're requesting
  * `params` - all record data&#x20;
  * `id` - record id
  * `title` - record title
  * `recordActions`- list all actions and their parameters available on this record
  * `bulkActions`- list of all bulk actions and their parameters available on this record

**Example**&#x20;

Endpoint: [https://demo.adminjs.co/admin/api/resources/User/records/63d3b2c982bf27f5606e44eb/delete](https://adminjs-demo.herokuapp.com/admin/api/resources/User/records/63d3b2c982bf27f5606e44eb/delete)

Response:&#x20;

```json
{
   "record":{
      "params":{
         "_id":"63d3b2c982bf27f5606e44eb",
         "firstName":"Admin Name",
         "lastName":"Admin Surname",
         "gender":"male",
         "email":"admin@adminjs.com",
         "isMyFavourite":true,
         "__v":0
      },
      "populated":{
         
      },
      "baseError":null,
      "errors":{
         
      },
      "id":"63d3b2c982bf27f5606e44eb",
      "title":"admin@adminjs.com",
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
