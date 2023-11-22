# Authentication

Version 7.4 of `adminjs` package introduces authentication providers which both simplify and extend authentication possibilities in your application.

Every authentication provider extends `BaseAuthProvider` class (exported by `adminjs` package). `adminjs` also exports `DefaultAuthProvider` which functions exactly the same as the current `authenticate` method.

### DefaultAuthProvider

As of version >=7.4.0 of `adminjs`, `DefaultAuthProvider` is an alternative to `authenticate` method. In the next major release, `authenticate` method will be removed in favour of auth providers.

<pre class="language-typescript"><code class="lang-typescript"><strong>import { DefaultAuthProvider } from 'adminjs';
</strong><strong>
</strong><strong>import componentLoader from '&#x3C;path to your component loader>';
</strong><strong>
</strong><strong>// Placeholder authentication function, add your logic for authenticating users
</strong>const authenticate = ({ email, password }, ctx) => {
  return { email };
}

const authProvider = new DefaultAuthProvider({
  componentLoader,
  authenticate,
});

// ...

// Express example, in other plugins the change is exactly the same
// "provider" should be configured at the same level as "authenticate" previously
  const router = buildAuthenticatedRouter(
    admin,
    {
      // "authenticate" was here
      cookiePassword: 'test',
      provider: authProvider,
    },
    null,
    {
      secret: 'test',
      resave: false,
      saveUninitialized: true,
    }
  );
</code></pre>

By migrating to class syntax, you should be able to modify any existing auth provider without making additional changes to your framework's plugin.

### BaseAuthProvider

```typescript
export interface LoginHandlerOptions {
  data: Record<string, any>;
  query?: Record<string, any>;
  params?: Record<string, any>;
  headers: Record<string, any>;
}

export interface RefreshTokenHandlerOptions extends LoginHandlerOptions {}

export class BaseAuthProvider {
  public getUiProps(): Record<string, any> {
    return {}
  }

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  public async handleLogin(opts: LoginHandlerOptions, context?: any) {
    throw new NotImplementedError('BaseAuthProvider#handleLogin')
  }

  public async handleLogout(context?: any): Promise<any> {
    return Promise.resolve()
  }

  public async handleRefreshToken(opts: RefreshTokenHandlerOptions, context?: any): Promise<any> {
    return Promise.resolve({})
  }
}
```

You can use `getUiProps` method to define configuration that will be sent to the frontend.

`handleLogin` method is neccessary to sign in your user and should return a user object or `null`, for example:

```typescript
  override async handleLogin(opts: LoginHandlerOptions, context) {
    const { data = {} } = opts
    const { email, password } = data

    return this.authenticate({ email, password }, context)
  }
```

`context` of `handleLogin` will always be an object containing Request/Response objects specific to your framework of choice.

`handleLogout` and `handleRefreshToken` are optional. `handleLogout` will be called before your user's session is destroyed in case you have to perform additional actions to log out the user. `handleRefreshToken` can be used to refresh your user's session if it's matched with an external authentication service. `handleRefreshToken` should return an updated user object (i. e. with a new access token). It is not used by default, but you can override `AuthenticationBackgroundComponent` component to periodically refresh your session.

```typescript
import { useCurrentAdmin } from 'adminjs';

const api = new ApiClient();

// ...

const AuthenticationBackgroundComponentOverride = () => {
  const [currentAdmin, setCurrentAdmin] = useCurrentAdmin();
  // ...
  // A part of your code responsible for refreshing user's session
  const requestBody = {};
  const response = await api.refreshToken(requestBody);

  const { data } = response;

  setCurrentAdmin(data);
  // ...
  return null;
}

export default AuthenticationBackgroundComponentOverride;
```
