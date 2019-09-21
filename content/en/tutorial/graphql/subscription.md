---
title: "GraphQL Subscription"
date: 2018-02-11T15:57:39-05:00
description: "A client to server integration pattern over websockets using graphql queries."
categories: [graphql, websockets, react]
keywords: [graphql, websockets, react]
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

## Introduction

GraphQL subscriptions are an easy and efficient way of providing live data to clients while maintaining all the numerous benefits of GraphQL.

The example located [here](https://github.com/networknt/light-example-4j/tree/master/graphql/subscription) provides a simple interaction model for a client written in react leveraging the apollo library, to be able to display live data updated in real-time (multiple sessions, for demonstration purposes).

## Getting Started

### JDK 8

The current example supports JDK 8 only. You can check your Java command-line version with `java -version`. 

### Bringing up light-4j server

From the root folder of the above example, open a new terminal, and follow along. To pull the dependencies and bring up the server in a single command, execute the following:

```
mvn clean package exec:exec
```

### Serving the UI

For development, we will be serving the UI separately from the data. So open a new terminal from within the `app` subdirectory of the same folder. Begin by installing all the `npm` dependencies:


```
cd app
npm i
```

Then bring up the server and serve the client files:

```
npm start
// navigate to localhost:3000
```

For the sake of the example, open two instances of `localhost:3000` in two different windows of the browser and navigate both to `localhost:3000/channel/1`.

If you type a message into the channel and click enter, the message is initially in the one window in grey. This signifies the optimistic response mechanism of apollo, which allows you to react in your window as if
the client instantly responded with the message being successfully added. After the message turns a lighter white, the real message has been sent from the server and should have also been issued to all other
observers of the channel. If you look at your another window, you should see the message you typed there as well.

## A Look Under the Hood

### Handling subscriptions on the client side.

In order to setup apollo to support subscriptions, a subscription client must be added to the network interface used to query GraphQL:

`App.js`
```
const networkInterface = createNetworkInterface({uri: 'http://localhost:8080/graphql'});
const wsClient = new SubscriptionClient('ws://localhost:8080/subscriptions', {
    reconnect: true
});
const networkInterfaceWithSubscriptions = addGraphQLSubscriptions(
    networkInterface,
    wsClient
);
const client = new ApolloClient({
    networkInterface: networkInterfaceWithSubscriptions
};
class App extends Component {
    render() {
        return (
            <ApolloProvider client={client}>
            ...
            </ApolloProvider>
        )    
    }
}
```

This provides apollo with the endpoints used for the different queries. Query/Mutation queries will be put to 
`http://localhost:8080/graphql` and WebSocket Subscription queries to `ws://localhost:8080/subscriptions`.

In the component that will be used to display real-time data, we attach to the componentWillMount hook and run the following:

`ChannelDetail.js`
```
class ChannelDetails extends Component {
    componentWillMount() {
        this.props.data.subscribeToMore({
            document: messagesSubscription,
            variables: {
                channelId: this.props.match.params.channelId
            },
            updateQuery: (prev, {subscriptionData}) => {
                ...
            }
        }
    }
}

const messagesSubscription = gql`
    subscription messageAdded($channelId: ID!) {
        messageAdded(channelId: $channelId) {
            channelId
            message {
                id
                text
            }
        }
    }
`;

export const channelDetailsQuery = gql`
    query ChannelDetailsQuery($channelId : ID!) {
        channel(id: $channelId) {
            id
            name
            messages {
                id
                text
            }
        }
    }
`;
```

The reason we have the `subscribeToMore` function available on the data property is from apollo returning the result of the `channelDetailsQuery` as an apollo object (and not just the json result). By running the `channelDetailsQuery` we get the current messages in the channel, and before mounting the component, we register to
listen for changes through the `messagesSubscription`.

### Handling subscriptions on light-4j

For the sake of the example, there are really only two fetchers we need to worry about on the server-side.
1. Mutation > addMessage
2. Subscription > messageAdded

This sample app was written using [graphql-java](https://github.com/graphql-java/graphql-java)'s IDL support
so the whole schema is available in `resources/config/subscription-schema.graphqls`

I suppose the fundamental concept to understand when handling GraphQL subscriptions on the server-side, is that
they return a `Flowable<T>` that other (likely mutation queries) publish to.

```
final static ChannelPublisher channelPublisher = new ChannelPublisher();
...
static DataFetcher messageAddedFetcher = dataFetchingEnvironment -> {
    return channelPublisher.getPublisher(...);
};
```

In the sample application, we have a single `final static hannelPublisher` that contains the field `final Flowable<MessageAddedEvent> publisher`. When we receive a subscription request on `messageAdded`, all we need to do is return that object. Granted, this is an extremely simplified case, and yours might require logic to return an existing `Flowable` from a map of them, or instantiate a new one.

```
public ChannelPublisher() {
    Observable<MessageAddedEvent> messageAddedEventObservable = Observable.create(messageAddedEventObservableEmitter -> {
        this.emitter = messageAddedEventObservableEmitter;
    });

    ConnectableObservable<MessageAddedEvent> connectableObservable = messageAddedEventObservable.share().publish();
    connectableObservable.connect();
    publisher = connectableObservable.toFlowable(BackpressureStrategy.BUFFER);
}
```

When we receive a call to Mutation > addMessage, we store the data in the cache, then add the message to the Observable of which the `Flowable` is tied to.

```
static DataFetcher addMessageFetcher = dataFetchingEnvironment -> {
    ...
    channel.addMessage(message);
    channelPublisher.onMessageAdded(new MessageAddedEvent(messageInput.getChannelId(), message));
    return message;
};
```

And as simple as that, we have GraphQL events continuously streamed to any listening clients over WebSockets.

Hope this helped! 