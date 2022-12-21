# Changing the form view



Default form view is based on CSS flex property with  `flex-direction: column`

If you would like to change this view AdminJS deliver layout option for actions (`new`, `edit` and `view`).

For example:

If our model has following fields: `name`, `surname`, `login`, `password` and we would like to have name and surname in one row and login and password below (also in one row) for new action we can use following example to achieve this

```javascript
    actions: [
      {
        name: 'new',
        layout: [
          ['@Header', { children: 'Enter user details' }],
          [
            { flexDirection: 'row', flex: true },
            [
              ['name', { flexGrow: 1, marginRight: '10px' }],
              ['surname', { flexGrow: 1 }],
            ],
          ],
          ['@Header', { children: 'Enter user credentials' }],
          [
            { flexDirection: 'row', flex: true },
            [
              ['login', { flexGrow: 1, marginRight: '10px' }],
              ['password', { flexGrow: 1 }],
            ],
          ],

        ],
      },
      // other actions
    ],
```

More detail information about layout structure you can find [here](https://github.com/SoftwareBrothers/adminjs/blob/master/src/backend/utils/layout-element-parser/layout-element.doc.md)
