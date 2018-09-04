# car-control (WIP)

## DB

DB structure

```
+-----------+                              +--------------+
|users      |                              |running       |
+-----------+                              +--------------+
|id         +---+------------------------->+userID        |
|login      |   |                      +-->+tripID        |
|password   |   |                      |   +--------------+
+-----------+   |                      |
                |  +---------------+   |
                |  |trips          |   |
                |  +---------------+   |
                |  |tripID         +---+
                +->+userID         |
                   |latitude       |
                   |longitude      |
                   |time           |
                   |odometerPicPath|
                   +---------------+

```

## Backend

The backend: 
- Stores images on the file system;
- Stores trips and image paths on a local db;
- Stores user credentials
- Retrieves trip and user data from db/filesystem

### Root

Serves a SPA that allows a user to view:
- User information;
- Trip data.

### User endpoint

#### POST

Creates a user for a given user and password.

#### GET returns list of users

```
[
    {
        userId: 1,
        login: joao.silva
    },
    {
        userId: 2,
        login: raissa.andrade
    }
]
```

### Login endpoint 

#### POST
Accepts POST requests with login and password and returns a userId credentials are valid.

### Trip endpoint

#### POST 
Accepts POST requests if a valid userId is provided. 

```
{
    userId,
    latitude,
    longitude,
    time,
    pictureBlob
}
```

#### GET with userId

Returns the start trip information for given userID:

```
{
    userId,
    latitude,
    longitude,
    time,
    picturePath
}
```

If there are no running trips for that userId, the endpoint returns an empty object:
```
{}
```

## Mobile App

React Native or jsonette.

### User Stories
- As a user I can log into the app.
- As user I can press start to begin a trip.
    - In order to start a trip, the app requests that I take a picture of the odometer.
    - A user can take a picture and accept a picture he took.
    - A user can reject a picture he took to try again.
    - A user can cancel the start operation.
- As a user, I can see a persistant notification once I start a trip.
- A user can press finish to end a trip.
    - In order to finish a trip, the app requests that I take a picture of the odometer.
    - A user can take a picture and accept a picture he took.
    - A user can reject a picture he took to try again.
    - A user can cancel the start operation.

### Behaviour

#### Main Screen

The main screen displays either a "Start trip" or an "End trip" button. To decide which button to display, the app makes a request to an authenticated endpoint to check if there as an ongoing trip for this user.

#### Start and End Trip
After the user takes a picture, the app POST's some information to an api endpoint:

```
latitude: 0.23327753,
longitude: -0.2566,
time: 112351231,
pictureBlob,
userId
```

### Requirements
- The app will only run if the user grants location, storage and camera usage permissions.
- The user must finish a previous trip before starting a new one.
- All strings displayed on the UI must be ready for localization.
