# How to write an addon?

With the introduction of AdminJS Marketplace, members of the AdminJS community can now publish their created plugins there, either for free or for a fee. In this article, we will describe how you can write your own plugin and what steps to take to make it appear in the AdminJS Marketplace at [https://cloud.adminjs.co](https://cloud.adminjs.co).

## Writing addons

There are various categories of addons that you can develop, but most fall into the following classifications:

1. **Adapters:** These are libraries that establish a connection between your AdminJS panel and a database, either directly or through an ORM/ODM.
2. **Plugins:** These libraries enable you to configure an AdminJS panel using different frameworks, facilitating the setup of API endpoints and views.
3. **Features:** Libraries categorized as features extend resource configuration to offer built-in functionalities, such as upload logic or generic actions.
4. **Themes:** Themes can range from a simple recoloring of the default AdminJS panel to a complete overhaul of its visual appearance.
5. **Authentication Providers:** Introduced in version 7.4.0 of the `adminjs` package, authentication providers simplify and broaden the possibilities for authenticating into AdminJS.

If you're keen on participating in the development of AdminJS, we recommend reviewing the following articles within this documentation:

{% content-ref url="basics/resource.md" %}
[resource.md](basics/resource.md)
{% endcontent-ref %}

{% content-ref url="basics/action.md" %}
[action.md](basics/action.md)
{% endcontent-ref %}

{% content-ref url="basics/property.md" %}
[property.md](basics/property.md)
{% endcontent-ref %}

{% content-ref url="basics/features/writing-your-own-features.md" %}
[writing-your-own-features.md](basics/features/writing-your-own-features.md)
{% endcontent-ref %}

{% content-ref url="ui-customization/writing-your-own-components.md" %}
[writing-your-own-components.md](ui-customization/writing-your-own-components.md)
{% endcontent-ref %}

{% content-ref url="basics/themes.md" %}
[themes.md](basics/themes.md)
{% endcontent-ref %}

{% content-ref url="basics/authentication/" %}
[authentication](basics/authentication/)
{% endcontent-ref %}

In case the articles above are not sufficient, feel free to reach out to us via our [official Discord server](https://adminjs.page.link/discord), and we will be happy to assist you.

## Guidelines

As addons are published on official AdminJS channels, there are several guidelines that each addon must adhere to in order to be published on the Marketplace:

1. Usage of Typescript to maintain the readability of the code,
2. Usage of ESLint to keep the code clean,
3. Compatibility with the latest versions of AdminJS packages and building ESM output,

Every submitted addons will also have to be reviewed and tested by an AdminJS developer.

## Submitting your addons

To publish your addon, please contact us through the [contact form](https://share.hsforms.com/1IedvmEz6RH2orhcL6g2UHA8oc5a) or [Discord](https://adminjs.page.link/discord). In the future, the process of adding addons will be automated.
