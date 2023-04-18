# Writing your own features

Features simplify writing code that is shared between your resources as they automatically handle merging of configuration.

A simple feature feature could be implemented as follows:

```javascript
const feature = (prevResourceOptions) {
  return {
    ...prevResourceOptions,
    actions: {
      ...prevResourceOptions.actions,
      edit: {
        ...(prevResourceOptions.actions && prevResourceOptions.actions.edit),
        //..
      }
      //..
    }
  }
}

export { feature }
```

As you can see, in the example above you have to take care of merging previous options, which could be problematic. Fortunately AdminJS gives you helper functions which help with this:&#x20;

* a factory function [buildFeature](https://github.com/SoftwareBrothers/adminjs/blob/master/src/backend/utils/build-feature/build-feature.ts),
* optional helper [mergeResourceOptions](https://github.com/SoftwareBrothers/adminjs/blob/master/src/backend/decorators/resource/resource-options.interface.ts) (when you need more control)

&#x20;This is how a feature could look like when [buildFeature](https://github.com/SoftwareBrothers/adminjs/blob/master/src/backend/utils/build-feature/build-feature.ts) is used:

```javascript
import { buildFeature, FeatureType } from 'adminjs';

import SomeModel from 'some-model.entity.js';

const someBeforeHook = () => { /* noop */ };

const myFeature = (config = {}): FeatureType => {
  // do something with your feature config?

  return buildFeature({
    actions: {
      edit: {
        before: [someBeforeHook],
      },
    }
  });
}

const SomeResource: ResourceWithOptions = {
  resource: SomeModel,
  features: [myFeature({})],
};

/* "someBeforeHook" will be used as a "before" hook in "edit" actions
for every resource where "myFeature" is added */
```
