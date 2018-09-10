# car-control (WIP)

## Backend

The backend:
- Stores images on the file system;
- Stores and serves user state for syncing.
- Serves a SPA for viewing data.

### DB | NoSQL.

```
#### User credentials
[login]:{
    login,
    pwdHash: sha256(password)
}

#### User state
[login]:{
    trips:[{
        plate
        start:{
            latitude,
            longitude,
            odometerPicHash,
            carPlate,
            time
        },
        finish:{
            latitude,
            longitude,
            odometerPicHash,
            carPlate,
            time
        }
    }]
}

#### Picture storage
[picHash]:'picture/path/in/filesystem/<picname>.jpg'

```

### Root

Serves a SPA that allows anyone to view:
- List of users
    - Row shows: | login |

Clicking on a user shows the user detail view:
- User login
- List of trips ordered by date, most recent first.
    - | Car plate |

Clicking on a trip shows the trip details:
- Start time
- Start position (map pin)
- Odometer picture
- End time
- Stop position (map pin)
- Odometer picture


### Endpoints

#### POST /user

Creates a user for a given user and password if the user is not taken.

#### GET /user

```
[
    'joao.silva',
    'raissa.andrade'
]
```

#### GET /user/:login

Returns latest state for a given user

#### POST /user/:login

Merges state provided in the BODY and saves it to the db.

#### POST /authenticate

Returns the user state if login and password provided in BODY are valid.

## Mobile App

React Native.

### User Stories
- As a user I can log into the app with my credentials.
- As user I can press start to begin a trip.
    - The start button is enabled once I enter a licence plate
    - In order to start a trip, the app requests that I take a picture of the odometer.
- As user can press finish to end a trip.
    - In order to finish a trip, the app requests that I take a picture of the odometer.

### Behaviour

#### Main Screen
1- Once a user logs in, the app syncs redux with the backend if internet connection is available.
2- The main screen displays either a "Start trip" or an "End trip" button. To decide which button to display, the app checks localStorage to see if there as an ongoing trip for this user.

#### Start and End Trip
After the user takes a picture, the app stores the picture locally and updates the state with information:

```
latitude: 0.23327753,
longitude: -0.2566,
time: 112351231,
localPicPath,
carPlate
```

The app persists and restores the redux state from local storage.

### Requirements
- The app will only run if the user grants location, storage and camera usage permissions.
- The user must finish a previous trip before starting a new one.
- The app will maintain a local state and sync with the endpoint if network is available.
- All strings displayed on the UI must be ready for localization.
