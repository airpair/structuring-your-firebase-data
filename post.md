## What is Firebase ?

Firebase is a trending Cloud based (<a href="http://en.wikipedia.org/wiki/Mobile_Backend_as_a_service">BaaS</a>) backend as a service which provides Real Time syncing with all the clients subscribed to the server at any given instance. Firebase supports multiple platforms/frameworks like AngularJS, EmberJS, Backbone.js, iOS7, OSX, Android and programming languages like Ruby, Node.js, Python, Java, Clojure, PHP, Perl & Javascript. Firebase relies on web sockets to update all the clients about any data changes instantly.

## Data In Firebase
### Storage Format
Data is stored in firebase as a large JSON document. It is the same case as it is done in most of the NoSQL database systems like MongoDB, Cassandra, CouchDB etc. The data is stored as a large objects which can hold key value pairs where value can be a string, number or another object. 

### Managing Data in Firebase

Firebase provides an excellent GUI to manage your data. The awesome part of it is that it is real real-time. You do not have to refresh your browser to see updated data as opposed to other DBMS GUIs. It shows the newly added, modified or deleted data objects using colors like green, yellow & red.

## Problem in Structuring your Data for Firebase
We all have been storing our data in a SQL based database and accustomed to relational data heirarchy from a long time. NoSQL world is quite different and finalizing the schema for your Firebase data will be a problem for you initially. I have seen a lot of queries and questions from people all around the world but no definitive guide or answer to this problem. In this article we will try to understand the problem in depth and find an elegant solution to deal with it.

### Example Data Schema 
In order to understand the problem, we will define the data schema we want to implement in Firebase for an imaginary application. I will take a common example of an app where we have three entities Users, Groups & Updates. The app is intended to send updates to a specific user or a Group consisting of multiple users.

<b>Properties or fields for Entities</b>

<img src="http://s1.postimg.org/6wp8wn4bj/Screen_Shot_2015_02_24_at_5_51_47_pm.png" width="340px">

### Relations or Entity Relationship Diagram

Requirement for Schema Relations:
1. Any user can be a part of a group.
2. Any group can include multiple users.
3. An update can be targetted for a single user or a particular group.

### Maintaining Relations in the SQL DBMS


We know that in order to implement relations in a SQL database we would need to create extra tables like 'user_belong_to_groups' with reference to user's primary key 'userid' and group's primary key 'groupid'. This table is very helpful during data retrieval because of SQL joins but in firebase you do not have Joins.

## Maintaining Relations in Firebase

In Firebase we cannot use extra object like 'user_belong_to_groups' to store the many to many relations because Firebase does not have any query language or any kind of joins. 

The next thought in our mind comes that we can nest data & objects. For example, I can have a Group object which can have a property users consisting an array of users and each user can have an array of updates. The data will look like something given below:

``` javascript,linenums=true
// JSON Structure for Data which has nested Objects
"data":{
  "groups":{
     "group1"{
            "group_name":"Administrators",
            "group_description":"Users who can do anything!",
            "no_of_users":2,
            "users":{
                "user1": {
                    "username":"john",
                    "full_name":"John Vincent",
                    "created_at":"9th Feb 2015",
                    "updates": {
                            "update1":{
                                "update_text":"New feature launched!",
                                "created_at":"13th Feb 2015",
                                "sent_by":"user2"
                        }
                    }
                },
                "user2": {
                    "username":"chris",
                    "full_name":"Chris Mathews",
                    "created_at":"11th Feb 2015"
                }
            }
        },
     "group2"{
            "group_name":"Moderators",
            "group_description":"Users who can only moderate!",
            "no_of_users":1,
            "users":{
                "user1": {
                    "username":"john",
                    "full_name":"John Vincent",
                    "created_at":"9th Feb 2015"
                }
            },
            ,
            "updates": {
                    "update2":{
                        "update_text":"Users should expect blackout tomorrow!",
                        "created_at":"19th Feb 2015",
                        "sent_by":"user1"
                }
            }
        }
    }
}

```

The problems with the above structure can be well analysed when you have a close look on the data. The issues with this type of structure for your data are
1. If the same user belongs to 2 groups we have to duplicate the complete user object which creates redundancy of data.
2. Accessing any user object directly using userid will be an issue.
3. It is very cumbersome to find all the groups a user belongs to. 
4. We have to create duplicate updates for showing it to multiple recipients.
5. We cannot set proper permissions in this kind of structure.

### Solving the Problem

In firebase, we have to adhere to some basic principles involved in managing NoSQL data. We can increase our application performance using these principles as it will make read & write operations swift.

#### 1. **Flatten (Denormalize)** Your Data  
We have to segment the data and make logical groups which may or may not represent the entities. We should think that how can we make accessing the objects fast. We should remember that in Firebase we can access the data easily using a URL. If I have Users object on the root object, I can easily access it like http://your-app.firebase.io/users.

For our example dataset, we will keep the Users & Groups data structure on the root objects itself. It will also help us in removing any data redundancy and ease of access to a particular object using id even. The data will look something like this:

``` javascript,linenums=true
{
  "users":{
    "user1":{
        "username":"john",
        "full_name":"John Vincent",
        "created_at":"9th Feb 2015"
    },
    "user2": ...,
    "user3": ...
  },
  "groups":{
    "group1":{
        "group_name":"Administrators",
        "group_description":"Users who can do anything!",
        "no_of_users":2
    },
    "group2": ...,
    "group3": ...
  }
}
```

#### 2. Storing **One to Many Relations** or Static Lists
If we know that some data will be unique to a particular object we can maintain it as a static list under that object. For Example: if we want to store the 'last_logins' for any user we can directly store it under the specific object instance. It will provide ease of access when we want to access 'last_logins' for a particular user.

```javascript,linenums=true
{
  "users":{
    "user1":{
        "username":"john",
        "full_name":"John Vincent",
        "created_at":"9th Feb 2015",
        "last_logins":{
            "l1":{ 
                "loginAt":"02/13/2015 03:40:40",
                "ip_address":"234.34.56.1"
            },
            "l2":{ 
                "loginAt":"02/12/2015 13:13:20",
                "ip_address":"156.12.36.1"
            }
        }
    },
    "user2": ...,
    "user3": ...
  }
  "groups": ...
 }
```


#### 3. Maintaining **Many to Many** Relations

We have already seen that we cannot nest Users in Groups as it will not represent many to many relation and will leave redundant data. We can create an index of groups under a specific containing only the keys of the groups which a user belongs to. This will enable us to easily fetch list of groups to which a user belongs to.

Similarly to complete the two way relation we should also have a property named members which will store the index of all the member users under it. The final data would look like the JSON document given below.

```javascript
{
  "users":{
    "user1":{
        "username":"john",
        "full_name":"John Vincent",
        "created_at":"9th Feb 2015",
        "groups":{
            "group1":true,
            "group3":true
        }
        "last_logins":...
    },
    "user2": ...,
    "user3": ...
  }
  "groups": {
     "group1"{
        "group_name":"Administrators",
        "group_description":"Users who can do anything!",
        "no_of_users":2,
        "members":{
            "user1":true,
            "user3":true
        }
      },
     "group2"{
        "group_name":"Moderators",
        "group_description":"Users who can only moderate!",
        "no_of_users":1,
        "members":{
            "user2":true
        }
      }
   }
 }
```

With this kind of structure, you should keep in mind to update the data at 2 locations under the user and group too. Also, I would like to notify you that everywhere on the Internet, the object keys are written like "user1","group1","group2" etc. where as in practical scenarios it is better to use firebase generated keys which look like **'-JglJnGDXcqLq6m844pZ'**. We should use these as it will facilitate ordering and sorting.

## Conclusion
If you follow some of the important principles mentioned in this article to structure your data, you will just love <a href="https://firebase.com">Firebase</a>. <a href="https://www.firebase.com/docs/web/guide/understanding-data.html">Here</a> is the ultimate guide to know everything about data on Firebase and how to use it correctly.

Now that you know how to structure your data in Firebase, wait for my next post on how to create a simple CRUD app using pure Javascript and Bootstrap.