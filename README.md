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
To begin the program, the name of an artist must first be entered. After the user has chosen an artist, the program calls the startRadio() function using the artist's name as a paramater. The startRadio() function starts by ending all current sessions in case a song is already playing, and then it uses the [Playlist Module] to create a static playlist based off of the chosen artist. The playlist module is provided by [The Echonest] and it creates a list of songs that are similar to the style of a certain artist. After creating a playlist based off of the chosen artist, the program loads the first song in the list into an audio player and begins playing it while it loads the second song in the list into a second audio player. The program uses these two audio players to play each song in the list, loading the next song in the list while the previous one plays. The author of the program used the following code to create a playlist with 30 songs based off of the artist the user chose:
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
The process() function handles most of the program's logic, including when to transition to the next song and how to transition. The function is executed once for each beat of the song, and as it goes through the song it keeps track of variables like waitTime and nextTime as well what state the audio player is in. The list of possible states includes 'intro', 'loading', 'ready-for-transition', 'transitioning', 'done', and 'no more'. By looking at the process() function you can see that there is no set time for a song to transition, instead it's based off of how long it takes for the next song to load. Once the next song to be played is fully loaded and at least 10 seconds of the first song has played, the trackReady() function sets the state of the audio player to 'ready-for-transition' which allows the process() function to begin playing the next song. The following is the boolean statement that determines whether or not the next song is loaded and ready to be played:
```python
if (slave && state === 'ready-for-transition' && totTime >= curSongTime - curSegueTime)
```
If the boolean statement is true, the state is changed to 'transitioning' and the program may begin handling the transition to the second song. This boolean statement is true when slave is set to some value (which is done when trackReady() is executed), when the state is set to 'ready-for-transition', and when the total time the song has been played is greater than or equal to 10 (curSongTime always equals 20 and surSegueTime always equals 10).

This resource answers question 2: how does Attention Deficit Radio decide when to change the song?

### 3. Mixing two songs together during a transition
After the state of the audio player has been set to 'transitioning', a conditional statement in the process() function will begin to mix the two songs while the old song transitions into the new song. The following is the code that causes the transition from the first song to the second:
```python
if (slave && state === 'transitioning') {
            var slaveVolume = curTransitionBeat / transitionBeats;
            var masterVolume = 1 - slaveVolume;
            var masterTempoFactor = (q.duration + periodDelta * curTransitionBeat) / q.duration

            masterPlayer.setGain(ngain(masterVolume));
            slavePlayer.setGain(ngain(slaveVolume));
            mixGauge.refresh(Math.round(slaveVolume * 100));


            masterPlayer.setSpeedFactor(masterTempoFactor);
            npGauge.refresh(Math.round(track.audio_summary.tempo / masterTempoFactor));
            var slaveDelay = slavePlayer.play(waitTime, slave.next());

            if (curTransitionBeat++ >= transitionBeats) {
                stop();
                releaseTrack();
                slave.setMaster(true);
                masterPlayer.setSpeedFactor(1);
                slavePlayer.setSpeedFactor(1);
                slave.start(nextDuration + waitTime);
                var swap = slavePlayer;
                slavePlayer = masterPlayer;
                masterPlayer = swap;
                mixGauge.refresh(0);
```
The program plays both songs together one beat at a time, changing certain aspects of each song as the transition progresses. When the transition occurs, the second song will start out playing quietly but it will slowly get louder over time while the first song lowers in volume. The songs will start out playing at the tempo of the first song, but the tempo will slowly raise or lower with every beat to adjust to the tempo of the second song. After the transition has occurred, the first audio player will have stopped playing audio and the second audio player will be playing the song normally. Finally, the first audio player and the second audio player will be swapped, making the first audio player the one playing the song again. Once the transition has finished, the state of the audio player will be 'intro' because the setMaster() function sets the state to 'intro' when the paramater is true, this will allow the next execution of process() to start from the beginning.

This resource answers question 3: how does Attention Deficit Radio mix two songs together during a transition?

### 4. Problems with this program
Although this program is interesting, it could definitely use some improvements. There are several problems with the program that limit its functionality, the first of these problems is the fact that the songs played are almost always the same. The first song played for any artist is always the same and the following songs are almost always the same. This is a problem with the javascript version of the playlist module, but it could still be fixed by creating additional playlists based off of the songs in the original playlist and taking songs from the resulting playlists. This would add additional variety to the songs played and it would cause the songs played to change with each run of the program.

Another major problem with this program is the fact that the transition between two songs isn't always smooth. Although some transitions are fairly smooth, others are incredibly jerky. An easy solution to this would be to make sure the tempos of two songs are similar if a transition is to occur. When one song with a tempo of 75 has to transition to a song with a tempo of 140, the necessary change in tempo with each beat is so extreme that it feels unnatural.

[Attention Deficit Radio]: http://static.echonest.com/AttentionDeficitRadio/
[Playlist Module]: http://developer.echonest.com/raw_tutorials/playlist_api/static.html
[The Echonest]: http://the.echonest.com/
[remix.js]: http://echonest.github.io/remix/js-tutorial.html
