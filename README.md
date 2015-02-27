# Problem
We want to look at [Attention Deficit Radio] and see how it makes the decisions it makes.

# Question
1. How does Attention Deficit Radio choose which song to go to next?
2. How does Attention Deficit Radio decide when to change the song?
3. How does Attention Deficit Radio mix two songs together during a transition?

# Resources
1. [Attention Deficit Radio]
2. [Playlist Module]
3. [The Echonest]
4. [remix.js]

### 1. Choosing the next song
To begin the program, the name of an artist must first be entered. After the user has chosen an artist, the program calls the startRadio() function using the artist's name as a paramater. The startRadio() function starts by ending all current sessions in case a song is already playing, and then it uses the [Playlist Module] to create a static playlist based off of the chosen artist. The playlist module is provided by [The Echonest] and it creates a list of songs that are similar to the style of a certain artist. The author of the program used the following code to create a playlist with 30 results based off of the artist the user chose.
```python
var url = 'http://developer.echonest.com/api/v4/' + 'playlist/static';
    $.getJSON(url, {
        api_key:apiKey,
        type:'catalog',
        artist:seedArtist,
        seed_catalog:cid,
        min_tempo:110,
        min_energy:.4,
        adventurousness:0,
        distribution:'focused',
        results:30,
        sort:'tempo-asc',
        bucket:['audio_summary', 'id:' + cid, 'tracks'],
        limit:true,
        _:cacheBuster()
    },
```

This resource answers the question 1: how does Attention Deficit Radio choose which song to go to next?

### 2. Deciding when to change songs
Most of the calculations of the program are in the process() function, this includes the calculations for when to play the next song. The function is executed once for each beat of the song, and as it goes through the song it keeps track of variables like waitTime and nextTime as well what state the audio player is in. The list of possible states includes 'intro', 'loading', 'ready-for-transition', 'transitioning', 'done', and 'no more'. By looking at the process() function you can see that there is no set time for a song to transition, instead it's based off of how long it takes for the next song to load. Once the next song to be played is fully loaded, the trackReady() function sets the state of the audio player to 'ready-for-transition' which allows the process() function to begin playing the song. The following is the boolean statement that determines whether or not the next song is loaded and ready to be played:
```python
if (slave && state === 'ready-for-transition' && totTime >= curSongTime - curSegueTime)
```
If the boolean statement is true, the state is changed to 'transitioning' and the program may begin handling the transition to the second song. 

This resource answers question 2: how does Attention Deficit Radio decide when to change the song?

### 3. Mixing two songs together during a transition
Using [remix.assemble] I can concatenate all the AudioDate objects in a list into a single AudioData object. After loading all the AudioData objects from a certain directory, I will be able to use the assemble module to combine all of the songs into one file that, when opened, will play all of the songs in the directory back to back.

### 4. Problems with this program
Encounters tracks it has no information for and plays them endlessly

Tempos don't match up

Always starts with same song

[Attention Deficit Radio]: http://static.echonest.com/AttentionDeficitRadio/
[Playlist Module]: http://developer.echonest.com/raw_tutorials/playlist_api/static.html
[The Echonest]: http://the.echonest.com/
[remix.js]: http://echonest.github.io/remix/js-tutorial.html
