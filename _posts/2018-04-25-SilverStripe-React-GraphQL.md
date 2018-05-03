# Getting Started with React, GraphQL, and SilverStripe

I completed my first project using React, GraphQL, and SilverStripe recently. Whilst I've worked with SilverStripe since version 2.1 and React for the past year or so, this was my first time combining the two - and my first time bringing GraphQL into the mix.

It wasn't as easy or as straightforward as I'd hoped, so here are a random collection of my thoughts and discoveries from doing this project. It's not a tutorial as such - let me know if there was more specific things you'd like me to go over.

## GraphQL is awesome! ##

I first came across GraphQL at the SilverStripe Conference in Wellington back in 2017. It sang out to me as the answer to all the issues with traditional REST APIs. I loved the declarative nature of GraphQL and how you get exactly what you ask for. If you're unfamiliar with GraphQL I'd highly recommend [this video](https://vimeo.com/206492203) by Aaron Carlino (aka Unclecheese)

### You need the GraphiQL dev tools ###

I don't know why they're not included but goddamn it you need them

```composer require silverstripe/graphql-devtools```

Then you can go to `http://yoursite.local/dev/graphiql` and see all the query types you can make.

### You need the silverstripe graphql documentation ###

silverstripe-graphql is iterating quickly. But the doco remains supurb. [This](https://github.com/silverstripe/silverstripe-graphql/blob/2/README.md) is really an invaluable resource.

## Authentication is a bit hard ##

This isn't directly related to GraphQL, but SilverStripe 4 has an all new authentication system. I'm told it's *wonderful* but it really does feel like a [dishwasher api](https://www.silverstripe.org/blog/cutting-through-the-noise-why-silverstripe-4-will-use-reactjs/#Diswashers_have_terrible_APIs_33).

Compare this login with SS3:

```php

$member->logIn();

```

With SS4
```php

Injector::inst()->get(IdentityStore::class)->logIn($member);

```

WTF is Injector? WTF is IdentityStore? I don't know and I don't care but it seems this is what I need to use to log in right now. ¯\\\_(ツ)\_/¯

Most of the time you'll probably just want to use session based authentication, so the most straightforward way to log in is to get the user to log in via normal SilverStripe methods first before showing the react app.

```html

<% if $LoggedIn %>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <script src="/resources/client/build/main.js"></script>
<% else %>
    $Content
    $Form
<% end_if %>

```

You could also log in via a GraphQL mutation. Create an extension for `Member` and scaffold something like this:

```php

<?php

namespace MyNameSpace\Extensions;

use SilverStripe\ORM\DataExtension;
use SilverStripe\GraphQL\Scaffolding\Interfaces\ScaffoldingProvider;
use SilverStripe\GraphQL\Scaffolding\Scaffolders\SchemaScaffolder;
use SilverStripe\Security\Authenticator;
use SilverStripe\Security\Member;
use SilverStripe\Security\Security;
use SilverStripe\Security\IdentityStore;
use SilverStripe\Control\Controller;
use SilverStripe\Core\Injector\Injector;

class MemberExtension extends DataExtension implements ScaffoldingProvider
{
  public function provideGraphQLScaffolding(SchemaScaffolder $scaffolder)
  {
    $scaffolder->mutation('login', Member::class)
      ->addArgs(['Email' => 'String!', 'Password' => 'String!'])
      ->setResolver(function($obj, $args, $context) {
        /** @var Security $security */
        $security = Injector::inst()->get(Security::class);
        $authenticators = $security->getApplicableAuthenticators(Authenticator::LOGIN);
        $request = Controller::curr()->getRequest();
        $member = null;
        $result = null;
        if (count($authenticators)) {
          /** @var Authenticator $authenticator */
          foreach ($authenticators as $authenticator) {
            $member = $authenticator->authenticate($args, $request, $result);
            if ($result->isValid()) {
              break;
            }
          }
        }
        if ($member) {
          Injector::inst()->get(IdentityStore::class)->logIn($member);
        }
        return $member;
      });
    return $scaffolder;
  }
}

```

Other options are to use jwt. Check out this [repo](https://github.com/Firesphere/silverstripe-graphql-jwt). It didn't suit me in this case as I needed the login to be the same whether accessing normal PHP pages or GraphQL.

## Use `create-react-app` ##

You don't need to know about webpack. Just use [`create-react-app`](https://github.com/facebook/create-react-app). Seriously.

## Tell SilverStripe to use your webpack when you're running it ##

Hat tip to @unclecheese right here, because this was really slowing me down.

Your PageController needs this:

```php

/**
  * Used for development to load webpack script as opposed to built js
  *
  * @return bool
  */
public function isWebpackRunning()
{
    return Director::isDev() && !!@fsockopen('localhost', 3000, $errorNumber, $errorMsg, 2);
}

```

Your template that has the React application needs this:

```html

<% if $isWebpackRunning %>
    <script src="//localhost:3000/static/js/bundle.js"></script>
<% else %>
    <script src="/resources/app/client/build/main.js"></script>
<% end_if %>

```

Now, if webpack is running it will connect directly to that, otherwise it will use your built bundle.

## Simplify deployment ##

There's probably something a lot smarter that you could do here, but I was in a hurry to ship and couldn't deal with parsing map files :P

Instead, I hacked the build script in `package.json` to move my files to static names.

```json

"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build && npm run build:clean",
    "build:clean": "cd build && move static\\js\\*.js main.js && move static\\css\\*.css main.css && rmdir /s /q static",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }

```
If you've got a better way I'm all ears!

## Handle javascript URLs nicely ##

In this project I had a multi page form, so it was important that users could come back to where they left off. For this I used `react-router` but of course you
need to tell SilverStripe to hand over to the page that renders your react app, even though this page doesn't exist in SilverStripe.

There are a few ways of going about this - for me I just made sure that the bulk of the react URLs were under a sub-url

```jsx

<Route exact path="/" render={props => <Home />} />
<Route exact path="/myapp/thing-one" render={props => <ThingOne} {...props}/>}/>
<Route exact path="/myapp/thing-two" render={props => <ThingTwo} {...props}/>}/>

```

and then in my `PageController`:

```php

<?php

//....
private static $url_handlers = [
   'myapp/$action' => 'index'
];

```

## Anything else? ##

Well, that's all I can think of that was a major blocker for me to get started. If you have any questions, feel free to hit me up. I'm `alt` on the [SilverStripe Slack](https://www.silverstripe.org/community/slack-signup), and [@altwohill](https://twitter.com/altwohill) on Twitter.