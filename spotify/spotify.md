
# Design Spotify
## Requirements 
- As a user i should have the ability to search for music
- As a user i should have the ability to play the music.
- As a user i should have the ability to add the music to my play list
- As a publisher should have the ability to publish the music to the service.
- As a user should have the ability to have subscription to listed to music on a continuous 
basis.

## Capacity estimation

## API's
- uploadService(app_key, albumType, artists, market, labels, name, releaseDate, tracks, restrictions, url);
- searchService(app_key, market, albumTitle, trackTitle);
- addToPlaylistService(app_key, playlistId, trackId);
- viewService(app_key, songId); //play_music
- paymentService(app_key, cardInfo);

## Data Modelling
![Datacbase Schema and Modelling](database_model.png)

## Implementation of distribution network
![Steps involved in distributing to audience from source](distribution.png)

## Overall architecture
![Overall architecture of Spotify](overall_architecture.png)

