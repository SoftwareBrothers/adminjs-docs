# Troubleshooting

Please read this article if you are having difficulties setting up your AdminJS application despite following installation instructions.

<details>

<summary>AdminJS login page or dashboard are not showing up after starting the application</summary>

As of version 6.0.0 and higher, there have been reports from community about AdminJS showing white, blank page after update or installation. This often results in one of the following errors appearing in your console:

```bash
  line-height: ${({ theme }) => theme.lineHeights.default};
                                                  ^
TypeError: Cannot read properties of undefined (reading 'default')
```

This error can be fixed by installing `styled-components` in your app with a version that matches the one used by `adminjs`.

```bash
$ yarn add styled-components@^5.3.5
```

\
Another error which often comes up is:

```bash
 throw Error( "Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for one of the followin
g reasons:\n1. You might have mismatching versions of React and the renderer (such as React DOM)\n2. You might be breaking the Rules of Hooks\n3. You
 might have more than one copy of React in the same app\nSee https://reactjs.org/link/invalid-hook-call for tips about how to debug and fix this prob
lem." );
            ^

Error: Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for one of the following reasons:   
1. You might have mismatching versions of React and the renderer (such as React DOM)
2. You might be breaking the Rules of Hooks
3. You might have more than one copy of React in the same app
```

This can be fixed by installing `react` and `react-dom` packages in your app:

```bash
$ yarn add react@^18.1.0 react-dom@^18.1.0
```

The above errors appear because of duplicated `styled-components`, `react` and `react-dom` packages in your `node_modules`, most likely due to external dependencies. An example is `styled-components` being installed under two different paths but containing the same package version:

```
<root>/node_modules/styled-components
<root>/node_modules/adminjs/node_modules/styled-components
```

These errors should disappear with time once more libraries update to React 18.

</details>

{% hint style="warning" %}
If you still cannot find a solution to your problem, please open an [issue in our GitHub.](https://github.com/SoftwareBrothers/adminjs/issues)
{% endhint %}

