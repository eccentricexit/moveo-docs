# Moveo

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

Accepts a user`s state and pictures provided in the BODY and saves the data to the db and file system.

#### POST /authenticate

Returns: 
- `true` if the crendentials are valid and there are no ongoing trips for the user.
- the state of the last trip if credentials are valid and there is a trip for this user.
- `false` if crendentials are invalid.

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
1- Once a user logs in, the app:
    - Initializes redux with state received from the backend (if any)
    - Replaces localstorage with the recent redux state built from the backend.
2- The main screen displays either a "Start trip" or an "End trip" button. To decide which button to display, the app checks localStorage to see if there as an ongoing trip for this user.

#### Syncing

The app should upload it's state to the backend once connectivity is accquired. This also means uploading pictures of trips.

#### Start and End Trip
After the user takes a picture, the app stores the picture locally and updates the state with information:

```
latitude: 0.23327753,
longitude: -0.2566,
time: 112351231,
localPicPath,
plate
```

The app persists and restores the redux state from local storage.

See format of the redux store must be equal to the state saved on the NoSQL db for easy serialization and deserialization, with the exception being `odometerPicHash` (which is only useful on the backend and meaninless on clients) and `localPicPath` which is used on clients only:

```
[login]:{
    trips:{
        [tripId]:{
            plate
            start:{
                latitude,
                longitude,
                localPicPath,
                time
            },
            finish:{
                latitude,
                longitude,
                localPicPath,
                time
            }
        }
    }
```

`localPicPath` should be removed before sending to the backend.

### Requirements
- The app will only run if the user grants location, storage and camera usage permissions.
- The user must finish a previous trip before starting a new one.
- The app will maintain a local state and sync with the endpoint if network is available.
- All code and comments in English.
- All strings displayed on the UI must be ready for localization.
