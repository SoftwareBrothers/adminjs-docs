# Features

This section contains detailed instructions on how to use AdminJS `features`.

Features are ready-made extensions for your resources. They extend existing resource options with predefined configuration and merge with your custom actions or properties configurations.

Features should be included in `features` section of your resource, example:

<pre class="language-typescript"><code class="lang-typescript"><strong>import { ResourceWithOptions } from 'adminjs';
</strong><strong>
</strong><strong>import User from './user.entity';
</strong><strong>
</strong><strong>const UserResource: ResourceWithOptions = {
</strong><strong>  resource: User,
</strong><strong>  options: {},
</strong><strong>  features: [someFeature({ /* feature config */ })],
</strong>};

export default UserResource;
</code></pre>

The most common use case of features is when you want some specific behaviour or configuration to be shared by multiple resources, for example: you may want to create a feature which logs changes to server's console.

If you'd like to learn how to write your own `features`, please visit:

{% content-ref url="writing-your-own-features.md" %}
[writing-your-own-features.md](writing-your-own-features.md)
{% endcontent-ref %}

## Official features

{% content-ref url="upload.md" %}
[upload.md](upload.md)
{% endcontent-ref %}

{% content-ref url="logger.md" %}
[logger.md](logger.md)
{% endcontent-ref %}

{% content-ref url="import-and-export.md" %}
[import-and-export.md](import-and-export.md)
{% endcontent-ref %}

{% content-ref url="password.md" %}
[password.md](password.md)
{% endcontent-ref %}

{% content-ref url="leaflet-maps.md" %}
[leaflet-maps.md](leaflet-maps.md)
{% endcontent-ref %}
