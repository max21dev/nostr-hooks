![nostr-hooks](https://socialify.git.ci/ostyjs/nostr-hooks/image?description=1&descriptionEditable=React%20hooks%20for%20developing%20Nostr%20clients.&font=KoHo&forks=1&issues=1&language=1&name=1&owner=1&pattern=Charlie%20Brown&pulls=1&stargazers=1&theme=Dark)

# Nostr-Hooks

React hooks for developing [Nostr](https://github.com/nostr-protocol/nostr) clients. It's simple yet intelligent.

![NPM Downloads](https://img.shields.io/npm/dt/nostr-hooks)

Nostr-Hooks is a stateful wrapper library of React hooks around [NDK](https://github.com/nostr-dev-kit/ndk), designed to simplify the process of interacting with the Nostr protocol in real-time web applications. It provides an easy-to-use interface with low bandwidth consumption and high performance, allowing developers to quickly integrate Nostr into their projects and build responsive, real-time applications.

## Supported by

<a href="https://opensats.org">
  <img src="https://opensats.org/logo.svg" alt="OpenSats" width="150px" />
</a>

## Features

- Provides high-level hooks to interact with the Nostr protocol, making it easy to integrate Nostr into React applications.
- Provides a single instance of Nostr pool for the entire application, which is reused by all components.
- Creates a single connection to each Nostr relay at a time and reuses it for all subscriptions, reducing network overhead.
- Automatically manages subscriptions from multiple components and delivers only the events that each component needs.
- Automatically batches multiple subscriptions from different components into a single subscription request, further reducing network overhead.
- Intelligently merges filters into a unique set of filters, reducing the load on the Nostr relays.
- Provides a built-in cache mechanism since version 1.1.
- Minimizes re-renders by updating only the events that have changed, improving application performance.
- Automatically cleans up subscriptions and garbage events when a component unmounts, preventing memory leaks.

<details>
  <summary>
    <b>
      Isn't nostr-tools or NDK enough? Why do we need Nostr-Hooks?
    </b>
  </summary>

Nostr-Hooks is not a replacement for NDK or nostr-tools. You may still need to install and use them in your application for some advanced low-level functionalities.
As you may know NDK is a powerful library (shout-out to [pablo](https://github.com/pablof7z)) with a lot of out-of-the-box features, like caching, batching, and merging filters. However, it's a **stateless** library and doesn't understand the React component lifecycle. This means that it's up to the developer to update the component state when new events arrive, and to unsubscribe from the subscription when the component unmounts. This can be a tedious and error-prone process, especially when scaling the application. Nostr-Hooks on the other hand, is a stateful wrapper library that manages the component state and subscriptions automatically, allowing the developer to focus on building and scaling the application.
Nostr-Hooks also provides a bunch of well-designed high-level hooks to interact with relays, so you don't need to worry about the low-level details any more.

</details>

## Usage

## Installation

```sh
npm install nostr-hooks
```

### First initialize the NDK instance and connect to it

```tsx
import { useNdk } from 'nostr-hooks';

const App = () => {
  const { initNdk, ndk } = useNdk();

  useEffect(() => {
    initNdk({
      // NDK Constructor Options
    });
  }, [initNdk]);

  useEffect(() => {
    ndk?.connect(); // This will also reconnect when the instance changes
  }, [ndk]);

  return <div>My App</div>;
};
```

> Calling `initNdk` and `ndk.connect` are mandatory to start using Nostr-Hooks.

### Subscribe to events

Here are some examples of how to use the `useSubscription` hook:

#### Example of basic usage:

```tsx
import { useSubscription } from 'nostr-hooks';

const MyComponent = ({ pubkey }: { pubkey: string | undefined }) => {
  const { events, isLoading, createSubscription, removeSubscription } = useSubscription(
    `${pubkey}-kind1`
  );

  useEffect(() => {
    if (!pubkey) return;

    const filters = [{ authors: [pubkey], kinds: [1] }];

    createSubscription(filters);

    return () => {
      removeSubscription();
    };
  }, [pubkey, createSubscription, removeSubscription]);

  if (isLoading) return <p>Loading...</p>;

  if (!events) return <p>No events found</p>;

  return (
    <ul>
      {events.map((event) => (
        <li key={event.id}>
          <p>{event.pubkey}</p>
          <p>{event.kind}</p>
        </li>
      ))}
    </ul>
  );
};
```

The `useSubscription` hook requires a `string` parameter as the subscription id. This id is essential for identifying and managing the subscription internally. Nostr-Hooks uses these subscription ids to categorize events throughout your application. It's recommended to define subscription ids based on your filter parameters, similar to a query key.

The hook returns an object with the following properties:

- `createSubscription`: A function that can be used to create a subscription with the provided filters and options.
- `removeSubscription`: A function that can be used to remove the subscription and clean up the events.
- `events`: An array of events that match the filters.
- `eose`: A boolean flag indicating whether the subscription has reached the end of the stream.
- `isLoading`: A boolean flag indicating whether the subscription is loading (no events yet, and no eose).
- `hasMore`: A boolean flag indicating whether there are more events available to fetch.
- `loadMore`: A function that can be used to fetch more events.

The `useSubscription` hook can be utilized across multiple components. Nostr-Hooks consolidates all subscriptions from various components into a single request, ensuring each component receives only the events it requires, based on their subscription ids.

### Publish new events

You can publish new events using the `NDKEvent` class and the `publish` method.

```tsx
import { NDKEvent } from 'nostr-dev-kit';
import { useNdk } from 'nostr-hooks';

const MyComponent = () => {
  const [content, setContent] = useState('');

  const { ndk } = useNdk();

  const handlePublish = () => {
    const event = new NDKEvent(ndk);
    event.content = content;
    event.kind = 1;

    event.publish();
  };

  return (
    <>
      <input type="text" value={content} onChange={(e) => setContent(e.target.value)} />

      <button onClick={() => handlePublish()}>Publish Note</button>
    </>
  );
};
```

### Fetch Profile for a user

The `useProfile` hook is used to fetch profile for a given user based on their `pubkey`, `npub`, `nip46 address`, or `nip05`. It returns the fetched profile.

```tsx
import { useProfile } from 'nostr-hooks';

const MyComponent = () => {
  const { profile } = useProfile({ pubkey: '...' });

  return (
    <div>
      <p>{profile?.displayName}</p>
      <p>{profile?.about}</p>
    </div>
  );
};
```

### Interact with Signer

You can leverage `useNdk` hook to interact with the signer.

```tsx
import { useNdk } from 'nostr-hooks';

const newSigner = new NDKNip07Signer();

const MyComponent = () => {
  const { ndk, setSigner } = useNdk();

  setSigner(newSigner); // this will keep the existing NDK instance and update its signer
};
```

> Direct usage of the `useSigner` hook might not be necessary. Refer to the following section for further details.

### useLogin: Login with different signers

Nostr-Hooks offers multiple login methods that automatically update the NDK instance with the new signer. These methods also utilize local storage to remember the login method, ensuring that users do not need to log in manually each time the page reloads or the app restarts.

We offer four methods for logging in with various signers, and one method for logging out:

- `loginWithExtension`: Login with Nostr Extension (NIP07).
- `loginWithRemoteSigner`: Login with Remote Signer (NIP46).
- `loginWithPrivateKey`: Login with Private Key.
- `loginFromLocalStorage`: Login from previously saved login method in local storage.
- `logout`: Logout.

```tsx
import { useLogin } from 'nostr-hooks';

const MyComponent = () => {
  const {
    loginWithExtension,
    loginWithRemoteSigner,
    loginWithSecretKey,
    loginFromLocalStorage,
    logout,
  } = useLogin();

  return (
    <>
      <button onClick={() => loginWithExtension()}>Login with Extension</button>
      <button onClick={() => loginWithRemoteSigner()}>Login with Remote Signer</button>
      <button onClick={() => loginWithSecretKey()}>Login with Secret Key</button>
      <button onClick={() => loginFromLocalStorage()}>Login from Local Storage</button>
      <button onClick={() => logout()}>Logout</button>
    </>
  );
};
```

#### Automatically login with previously saved login method:

You can use `loginFromLocalStorage` to automatically login with previously saved login method in local storage when the component mounts.

```tsx
import { useLogin } from 'nostr-hooks';

const MyComponent = () => {
  const { loginFromLocalStorage } = useLogin();

  useEffect(() => {
    loginFromLocalStorage();
  }, [loginFromLocalStorage]);
};
```

### Getting the Active User Profile

The `useActiveUser` hook allows you to retrieve the profile of the currently active user, utilizing the provided NDK instance and its signer.

```tsx
import { useActiveUser } from 'nostr-hooks';

const MyComponent = () => {
  const { activeUser } = useActiveUser();

  if (!activeUser) return <p>Not logged in</p>;

  return (
    <div>
      <p>{activeUser.pubkey}</p>
    </div>
  );
};
```

> If the user is not logged in, the `activeUser` will be `undefined`.

## Contributing

We welcome contributions from the community! If you'd like to contribute to Nostr-Hooks, please refer to the [CONTRIBUTING.md](https://github.com/ostyjs/nostr-hooks/blob/master/CONTRIBUTING.md) file in the project's GitHub repository.

> You can also consider contributing to [NDK](https://github.com/nostr-dev-kit/ndk).

## Donations

If you'd like to support the development of Nostr-Hooks, please consider donating to the developer.

- ⚡ Zap sats to [sepehr@getalby.com](sepehr@getalby.com)

> You can also consider supporting the [NDK](https://github.com/nostr-dev-kit/ndk).

## License

Nostr-Hooks is licensed under the MIT License. For more information, see the [LICENSE.md](https://github.com/ostyjs/nostr-hooks/blob/master/LICENSE.md) file in the project's GitHub repository.

## Contact

If you have any questions or concerns about Nostr-Hooks, please contact the developer at [npub18c556t7n8xa3df2q82rwxejfglw5przds7sqvefylzjh8tjne28qld0we7](https://njump.me/npub18c556t7n8xa3df2q82rwxejfglw5przds7sqvefylzjh8tjne28qld0we7).
