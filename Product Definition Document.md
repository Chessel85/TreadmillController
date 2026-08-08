# Product Definition Document: - Treadmill controller 

## 1. Overview
Treadmill Controller is a web page hosted on a Github repo used just by myself to control my JTX Sprint treadmill and to upload a run to my Strava account.  It supports a treadmill session conducted in manual mode where the web page just logs the activity for later uploading to Strava, and also supports reading a session defined by a text file defining duration, speed and incline changes.

## Objectives 
The primary objective is to allow time spent on the treadmill to be accurately logged against my Strava account as activity so myself and friends can see my treadmill workouts.  The secondary objective is to allow programming of a session so a predetermined run can be carried out without having to manually adjust the treadmill once the programme is loaded and started.

## assumptions

* Using an iPhone 13 running up to date ios version.
* Bluefy browser is available 
* Github repo is available and also set up as a web server.
* Treadmill is a JTX sprint 6.
* Main user is blind and uses VoiceOver on their iPhone 

## Roles and Requirements 

### 1. User Roles 

The following roles are defined for operating the application:

| Role | Description & Primary Goal |
| :--- | :--- | 
| Visually Impaired  runner  | A visually impaired runner using Voice Over on their iPhone using their JTX Sprint treadmill  |

### 2. Functional Requirements 

The following table lists requirements along with acceptance criteria.

| Ref | Category | User Story | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| 1 | Joining | As a VI user I need to connect to my JTX Sprint treadmill | * The application lists available treadmills to connect to<br> * The treadmill is easy to select and initiate joining to |
| 2 | Modes | As a VI user I want to select whether to do a manual run where the controls on the treadmill are used to control incline and speed, or to follow a plan where the incline, speed and duration are controlled by a plan  | * Selection of modes is easy but can only be returned to once a session has been cancelled or ended |
| 3 | starting | As a VI user I need to be in control of when the session starts | * Once manual mode is selected, a clear 'Start' button is the point at which logging of activity begins. <br> * For a planned run, a start button is available once the plan has been selected at which point logging of activity begins. |
| 4 | Plan selection | As a VI runner I want to be able to pick from a list of available plans | * Plans consist of simple text files that are uploaded by the user from their PC |
| 5 | Logging | As a VI runner I want the application to log my run capturing incline and speed and the duration of these as they change through the course of the session | * * Strava probably wants to know just the time and distance of the activity but a more detailed breakdown of a run could be made available to the user |
| 6 | Pausing | As a VI runner I want to be able to pause and resume logging | * Pausing is on the app and does not change the treadmill behaviour |
| 7 | finishing | As a VI user I want to be able to end a manual session and also end a planned session prematurely, as well as a planned session ending naturally as it is completed. | * Finishing does not change the treadmill speed and incline as this remains the responsibility of the runner. |
| 8 | Plans | As a VI runner I want a planned run to mean the application controls the treadmill incline and speed according to the plan although the user can still make manual adjustments directly on the treadmill | * There are no incline and speed controls on the application |

| 9 | strava | As a VI runner I want a finished session to be uploaded to my Strava account | * Ideally with minimal authentication |
| 10 | Audio (Should) | As a VI runner I want to listen to Spotify over my Bluetooth headphones while using the app | * The app does not provide in-app music controls — Spotify is controlled independently <br> * User starts Spotify playback before launching Bluefy, as an operational habit <br> * Simultaneous BLE (treadmill) and Bluetooth Classic (headphone audio) connections are supported by iOS; no special handling required in-app |
