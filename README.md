This repository demonstrates how do I benefit from tiny functional programming tools to 
have my code more declarative.

I'll code a simple app that is supposed to fetch the following data from an ad-hoc API and list
all of them as HTML;

```js
// => /users.json
        [3, 7, 19, 23, 27]
  
// => /users/7.json
        { id: 7, name: 'Smith', age: 21, posts: [3, 11, 12], photos: [19, 23, 39] }
    
// => /posts/11.json
        { id: 11, title: "Hello World", content: "lorem ipsum sit dolor amet" }
    
//  => /photos/19.json
        { id: 19, path: "http://photos.foobar.com/19.jpg" }
```

Let's think about the dependencies we need; a function to query JSON API?

```js
var getJSON = require('get-json');
```

At the first step, we need the list of users so we'll send a request to `/users.json`. 
It's a simple one, so we can do partial application on `getJSON`:

```js
var partial = require('new-partial');

var userIds = partial(getJSON, '/users.json');
```

`userIds` above is a partial application, just another async function, nothing special. It'll send a request to /users.json and
pass you the data once you call it with a callback.

Since we now have the list of users to get, let's get the user data:

```js
var joinParams = require('join-params');

var user = joinParams(getJSON, "/users/{0}.json")
```

This time, we used `join-params` for getting the partial application of `getJSON`. 
[join-params](http://npm.im/join-params) is a fork of [new-partial](http://npm.im/new-partial)
that lets you join the parameters in a single, formatted parameter. See its docs for details.

Now we have the list of users and a singular user implementation. We'll use partial and [map](http://npm.im/users) together to
define the plural form of `user`;

```js
var map = require('map');

var users = partial(map, user);
```

At this step, we have user ids and a collection that implemens group of users. All we need is to combine these two together, using
a function composition library, [comp](http://npm.im/comp):

```js
var comp = require('comp');

var allUsers = comp(userIds, users);

allUsers(function(error, allUsers){
        
        allUses[0].name, allUsers[2].age
        // => Smith, 23
        
})

```

Another example, partial application of `users` ?

```js
var adminIds = [3, 7, 9];
var admins = partial(users, adminIds);

admins(function(error, admins){
        
        admins[0].name, admins[1].age
        // => "Smith", 21
        
})
```

Until this point, we're able to get all users with their data excluding posts and photos, since they require
separate API calls.

===============================

Dependencies:

```js
var andThen    = require('andthen'),
    getJSON    = require('get-json'),
    joinParams = require('join-params')
    map        = require('map'),
    partial    = require('new-partial');
```

Implementation:

<a name="impl"></a>
```js
var getUserIds      = partial(getJSON, '/users.json'),

    getUser         = joinParams(getJSON, '/users/{0}.json'),
    getUsers        = partial(map, getUser),
    
    getAllUsers     = andThen(getUserIds, getUsers),
    
    getPost         = joinParams(getJSON, '/posts/{0}.json'),
    getPosts        = partial(map, getPost),
    
    getPhoto        = joinParams(getJSON, '/photos/{0}.json'),
    getPhotos       = partial(map, getPhoto);
    
    getProfile      = andThen(getUser, 'posts', getPosts, 'photos', getPhotos),
    getProfiles     = partial(map, getProfile),
    getAllProfiles  = andThen(getUserIds, getProfiles);
```

