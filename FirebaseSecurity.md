# Firebase Authentication and Authorization

## Authentication with Firebase
To be discussed...

## Authorization in Firebase Database
![Screenshot of Firebase Database](https://a43d55f6a02c4be185ce-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/Firebase/firebase-database-console.png)
As shown in the image above, the two tabs, "DATA" and "RULES", are the most important ones in Firebase Database.
"DATA" is where we store all of our app's data in a single tree-like document. 
"RULES" is where we define who has access to what data in our "DATA".
This is what the defualt rules look like. And it's basically saying that all authenticated users have acess to
read and write all of our data in the database.
```javascript
{
    "rules": {
        ".read": "auth != null",
        ".write": "auth != null"
    }
}
```

For basic info about the rules with Firebase Database security, read:
https://firebase.google.com/docs/reference/security/database/

### Demostration - Football Game
Suppose we are going to build a database for a football app with information for different games.
We want our admins to have acess to write data, and we want to implement authorization for our authenticated users
for their access to the data in our database.

The rules for our app is that all the logged in users have access to the meta-data of a game, like date, opponent, 
and whether the game is public, and this isPublic says whether that the content of the game can be visited without a ticket or not. 
If the game is not public, we only the ticket holders visit the game (read the game content). Also, the ticket holders 
have write access to his or her own ticket only.

This is the data in our database: 

```javascript
{
    "admin": {
        "admin id string 1": {
            "name": "Josh Nash"
        },
        "admin id string 2": {
            "name": "Ryan Di"
        }
        // ...
    },
    "games": {
        "random push key 1" : {
            "metadata" : {
                "date": "date 01",
                "opponent": "Chelsea",
                "isPublic": true // this determines whether the game content is public or not
            },
            "gameContent" : {
                /*
                content that only authorized users can have access to
                just like for a football game, only those with tickets are granted
                access to the stadium
                */
            },
            "tickets": {
                // ... 
            }
        }, 
         "random push key 2" : {
            "metadata" : {
                "date": "date 02",
                "opponent": "Manchester City",
            },
            "gameContent" : {
                // ...
            },
            "tickets": {
                // this contains all the users whos have access to the gameContent
                "user id string 1": true,
                "user id string 2": true
            }
        }
    }
}
```

We see that game one is public and thus its content can be read by all users who are authenticated.
However, game two is not public, thus we only want verified users to be able to see the content.

In our rule declartion:

A very important thing about Firebase Database rules is that firebase cascades all rules in that location and the 
child nodes of that location.

```javascript
{
    "rules": {
        "games": {
            // Naming with a dollar sign like $gameId in Firebase is called a wildcard.
            // This refers to all the keys just nested within "games" to refer to each game though with
            // a different id
            "$gameId": {
                "metadata": {
                    ".read": true,
                    /* 
                      auth.uid is the id of the current authenticated user
                      Again, write rules cascade.
                      Authenticated admin users have write access to all the data at current location
                      and the child nodes of the current location.
                      To change the write access or validate the input data, use ".validate" rule.
                    */
                    ".write": "root.child('admins').hasChild(auth.uid)",
                    ".date": {
                        ".validate": "newData.isString()"
                        // and we can use regular expression as well
                        // ".validate": "newData.val().matches(/^\\d{2}-\\d{2}-\\d{4}$/)"
                    },
                    "opponent": {
                        ".validate": "newData.isString()"
                    },
                    "isPublic": {
                        ".validate": "newData.isBoolean()"
                },
                "$other": {
                    // this is saying that all writes will be denied for other nodes under each game node
                    ".validate": false
                }
                "gameContent": {
                    ".read": "data.parent().child('tickets').hasChild(auth.uid) || data.child('isPublic').val() === true"
                },
                "tickets": {
                    "$uid": {
                        // User-based security
                        ".read": "auth.uid === $uid",
                        ".write": "auth.uid === $uid"
                    }
                }
            }
        }
    }
}
