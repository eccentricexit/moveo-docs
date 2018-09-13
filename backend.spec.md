# moveo backend

> Server of static SPA and api for creating users, fetching trip information and downloading images.

See frontend.spec.md for details on the SPA.

> **Important**: As of version 0.1.0 none of the endpoints are authenticated. The `/authenticate` endpoint is only a convenience for clients.

- [moveo backend](#moveo-backend)
    - [API](#api)
        - [GET `/`](#get)
        - [GET `/user`](#get-user)
        - [POST `/user`](#post-user)
        - [POST `/authenticate`](#post-authenticate)
        - [GET `/user/:login`](#get-userlogin)
        - [GET `/picture/:hash`](#get-picturehash)
        - [POST `/user/:login`](#post-userlogin)
    - [DB](#db)
        - [users](#users)
        - [userState](#userstate)
        - [picturePath](#picturepath)

## API

All endpoint paths should be prepended with `/api/v1`, with the exception of the root serving static content.

A GET request to an api endpoint would look like: `curl http://localhost/api/v1/user`.

### GET `/`

Serves static content available on a folder.

### GET `/user`

returns: a JSON object with an array of user objects:

```
{
    status: 200,
    response: [
        {
            login
        }        
    ]
}
```

Example of response:
```
{
    status: 200,
    response: [
        {
            login: 'joao.silva'
        },
        {
            login: 'raissa.andrade'
        }
    ]
}
```

### POST `/user`

`curl -d "login=joao.silva&pass=12345" http://localhost/api/v1/user`

Reads parameters from the body, `application/x-www-form-urlencoded` and saves to the db if possible.

parameters:
``
login,
password
``

> password.length must be >= 6 digits.

If the login is taken, returns:
```
{
    status: 400,
    response: `login ${login} is not available
}
```

If the login is available, stores the following information on the `user` document:

```
[login]:{
    login,
    pwdHash: sha256(password|salt),
    salt
}
```

where:
- `salt` is a large number generated with a cryptographically secure random number generator.
- `pwdHash` is the sha256 hash of the password concatenated with the salt.

once the db responds that it stored the data successfully, the endpoint should return:

```
{
    status: 200,
    response: { 
        login: [login]
    }
}
```

### POST `/authenticate`

`curl -d "login=joao.silva&pass=12345" http://localhost/api/v1/user`

Reads parameters from the body, `application/x-www-form-urlencoded`.

parameters:
``
login,
password
``

The endpoint fetches the `salt` for the provided `login`, available on the `user` document and calculates `calculatedPwdHash` as follows:

```
calculatedPwdHash = sha256(password|salt)
```
where | is concatenation.

if `calculatedPwdHash` is different than `pwdHash` stored on the database for that user, returns:

```
{
    status: 404,
    response: 'invalid login and password combination'
}
```

otherwise the endpoint returns:

```
    status: 200,
    response: { 
        [login]: {
            login
        }
    }
```

### GET `/user/:login`

`curl http://localhost/api/v1/user/joao.silva`

returns: a JSON object for the user in the format
```
{
    [login]:{
        trips:{
            [tripId]:{
                plate
                start:{
                    latitude,
                    longitude,
                    odometerPicHash,
                    time
                },
                finish:{
                    latitude,
                    longitude,
                    odometerPicHash,
                    time
                }
            }
        }
    }
}
```
where
- `tripID` is a uuid v4 generated on the client.
- `odometerPicHash` is the md5 hash of the picture taken.

An example of a returned object:

```
{
    'joao.silva':{
        trips:{
            '339cdd3b-1746-479d-ac22-4b40a6e5c4a4':{
                plate:'GHG-3324'
                start:{
                    latitude:0.023111031
                    longitude:0.0123322331
                    odometerPicHash:'08afd6f9ae0c6017d105b4ce580de885',
                    time:1536843597
                },
                finish:{
                    latitude:'0.0232114411'
                    longitude:'0.0123361234'
                    odometerPicHash:'de0729c647b7f6d007466d233a6037fd',
                    time:1536846186
                }
            }
        }
    }
}
```

### GET `/picture/:hash`

`curl http://localhost/api/v1/picture/de0729c647b7f6d007466d233a6037fd`


if `hash` does not exist in `picturePath` document, returns:
```
{
    status: 404,
    response: 'invalid login and password combination'
}
```

otherwise the endpoint returns the image.

### POST `/user/:login`

Accepts `multipart/form-data` to upload a user's trip data along with two images.

```
This endpoint:

1. Parses the stringified JSON object on `body.trip`;
2. Builds an object from the trip data that includes the pictures' hashes;
3. Updates the `userState` document with the object created in `4`;
4. Saves the picture to the file system;
5. Inserts an entry on the `picturePath` document, mapping a picture's hash to it's path on the file system;
```

> Tip: Use [multer](https://github.com/expressjs/multer) to parse the request: 
- `body.files` is an array with two images, the first being the start picture and the second being the end picture.
- `body.trip` is a stringified JSON object with information about the trip as follows:


1. The parsed JSON object looks as follows:

```
{
    [tripId]:{
        plate
        start:{
            latitude,
            longitude,                
            time
        },
        finish:{
            latitude,
            longitude,
            time
        }
    }
}
```

1. Once all the endpoint has both files, it calculates the md5 hash for each picture and stores them on an object:

Example:
```
trips[tripId].start.odometerPicHash = md5(body.files[0])

and

trips[tripId].start.odometerPicHash = md5(body.files[1])
```

3. Update the `userState` document on the database for that user with the new trip data:

```
[login]:{
    trips:{
        [tripId]:{
            plate
            start:{
                latitude,
                longitude,
                odometerPicHash,
                time
            },
            finish:{
                latitude,
                longitude,
                odometerPicHash,
                time
            }
        }
    }
}
```

4. Save the pictures to the file system. The path is as follows:

`/login/${tripId.slice(0,8)}/${time}.jpg`

5. Insert entries on the `picturePath` document for each picture, mapping their md5 hash to the path.

Example:
```
'08afd6f9ae0c6017d105b4ce580de885':'/joao.silva/478c51fe/1536843597.jpg'
'de0729c647b7f6d007466d233a6037fd':'/joao.silva/de0729c6/1536846186.jpg'
```

## DB

The database is a NoSQL database with 3 documents:

- `users`: Stores user credential data.
- `userState`: Stores user state information.
- `picturePath`: Maps a picture`s md5 hash to its location.

### users
[login]:{
    login,
    pwdHash,
    salt
}

### userState
[login]:{
    trips:{
        [tripId]:{
            plate
            start:{
                latitude,
                longitude,
                odometerPicHash,
                time
            },
            finish:{
                latitude,
                longitude,
                odometerPicHash,
                time
            }
        }
    }
}

### picturePath
[picHash]:'picture/path/in/filesystem/<picname>.jpg'

```