### Adding Musical Dimension to a Game: The Making of Line Out's Rhythm Mechanics

I had a lot of fun implementing the rhythm mechanics in Line Out, and by "fun" I of course mean this both genuinely and in a sarcastic manner. My first time making a rhythm game was no picnic, and from what I've heard, I'm not alone in that camp. It took me several iterations and hours of reworking the code until I had a stable, well-functioning rhythm manager. The first iteration was done within 9 weeks, which was the amount of time the game was meant to be completed in, so at that time it was hardly stable.

I made the decision to keep poking at what I already had instead of starting from scratch when the team decided we wanted to continue working on the game after the 9 weeks was done. This was maybe not the best decision, since the existing rhythm manager was a total mess and old forgotten components not used anymore still hide away within it. It's become a joke among the team, albeit there is truth in the name, that my role is "Rhythm Manager Engineer" simply due to the amount of times "fix the rhythm manager" has reappeared on my to-do list. Retrospectively, its evolution into what it is now taught me a lot when it comes to improving structure and writing code more efficiently, and additionally it's inspired me to keep making rhythm games because I know I can do better now.

Now, music in games has always been important to me, but there's something unexplainable that comes over me sometimes when a game integrates its soundtrack as more than just background noise. Even if it's something as trivial as UI components pulsing to the beat of the music. [Here's an example of what I mean.](https://www.youtube.com/watch?v=OBQE_TNI7zw) It's so simple, but it really pulls together the presentation. Maybe it's my old musical theater side that loves a presentation with many moving parts that all listen to each other. After all, audio is too often neglected in the game development process.


### Writing the music

The music I wrote for Line Out I wanted to feel free and adventurous like if I were a dog running through Swedish Lapland. The first track I wrote that I intended to put in a level reflected this vision the most. But upon showing the rest of the team, the feedback I received was that it should have higher tempo and more motion, like a Mario Kart level (Mario Kart being the main comparison title we used for reference throughout development). This song became the main theme because it still fits the idea, but for a level, the music needed to be piu mosso, fast and chaotic, like the game is. 

<div style="text-align: center;">
<video controls width="800">
<source src="{{ "/_media/Lapland'sFikaDeliveryDogs.mp4" | relative_url }}" type="video/mp4">
</video>
</div>


While still thinking of freedom and adventure, something I kept in mind while writing all the music was the idea of up and down, falling apart and back together again, tension and release, and also growth. Because even though the game is about chaos, it's also about working together with your team, and there will be times when you flow, and times when you tangle, and the longer you work together, the better you get at it. I wanted the different parts to represent the different players on the team. At times they may be doing completely different things, and other times will be in perfect harmony or dissonant harmony. The calls and responses and countermelodies are the players communicating and bouncing off one another in the game, just as musicians do when they play together. These ideas along with inspiration from Sonic, Splatoon, and Mario Kart, of course, is what shaped the soundtrack.



### Creating custom soundtracks

In order for the rhythm manager to keep the beat, it needs information directly from the music, so I created a Soundtrack class which holds the BPM, how many bars in the track, how many beats per bar, the current playback position, and other variables. In the game, when a rhythm event is triggered, the sequence you must perform depends on which part of the song is playing. This means that the soundtrack must also have a list of the sequences as well as how many bars each one lasts for.

One track in particular has several time signature changes, so the soundtrack must also have a list of these time signatures and how many bars each one lasts for. Implementing this was worth it for the flexibility and reusability, even if it is just one track for now.


<div style="text-align: center;">
<video controls width="800">
<source src="{{ '/_media/DankRoastMIDI.mp4' | relative_url }}" type="video/mp4">
</video>
</div>



### Embedding the mechanics cleanly

I don't consider Line Out to be much more of a "rhythm game" as [Mother 3](https://www.youtube.com/watch?v=4l6DkXztKeE&t=6s) is. The mechanics are there if you want, but it is not the main thing players think about while playing. However, the rhythm aspect does contribute to the core pillars of the game, which include collaboration/synchronization between players, so I do still consider it a key component to the game as a whole. Additionally, the game is targeted to be a chaotic couch co-op party game, so the mechanics must be accessible to anyone, including players who are not rhythmically inclined. 

Figuring out how to blend the rhythm mechanics into a chaotic and mainly physics-based game in a way that feels intuitive was challenging enough as it was, but then introducing how it works to new players in a way that does not slow the flow was a whole other obstacle. Holding the players' hands through how it works in the tutorial level sort of defeats the idea of timing-based mechanics and also inhibits the flow of the game. The team was adamant that the game should feel fast-paced and anything the slows down the gameplay loop is unnecessary friction that turns players off. But no tutorial at all leaves players confused at what they're supposed to do during a rhythm event. We tested and iterated on the presentation and UI of the rhythm mechanics several times with as many playtesters who had never seen the game as we could find by bribing them with coffee and cookies. 

After getting my heart broken many times over by playtesters not getting the hang of the mechanics and finding all kinds of bugs in the rhythm manager, we finally got to a place where the mechanics are introduced smoothly and unpunishingly without disrupting flow. On the countdown before each level, there is a simple initial rhythm event which utilizes (and for first time players introduces) the mechanic. Depending on how they perform, they receive a speed boost as the level begins, similar to in Mario Kart where pressing the button at the right time before the timer begins gives a speed boost. 



<div style="text-align: center;">
<video controls width="800">
<source src="{{ '/_media/InitialRhythmEvent.mp4' | relative_url }}" type="video/mp4">
</video>
</div>


In our tutorial level, we also have pre-placed billboards with textual explanations of the mechanics for players who wish to take the time to read them. Those who want to keep moving can choose to run straight past them. This way, there is no disruption or annoying tutorial popups that take away from the flow of the game. It is up to the player to decide if they want to learn the mechanics through simply playing or by reading explicitly how it works.




### Rhythm calibration

Because some audio output devices have a considerable amount of delay, it was crucial that I implemented a way for players to be able to calibrate the timing of the rhythm. 

Here's a clipping from the Soundtrack class which updates the playback position while taking into account the audio calibration settings:
```cs
  _curPlaybackPosition = GetPlaybackPosition() + (float)AudioServer.GetTimeSinceLastMix();
  _calibratedPlaybackPosition = ((_curPlaybackPosition + _audioCalibrationOffset) + (float)Stream.GetLength()) % (float)Stream.GetLength();
  _metDelta = _calibratedPlaybackPosition - _metDelta;
```
I actually implemented two modes of calibration: one for audio latency, and one for video latency, to calibrate the visual cues as well. The calibration not only takes into account device latency, but also player reaction timing. Rhythm calibration settings are a must-have for any rhythm game.


<div style="text-align: center;">
<video controls width="800">
<source src="{{ '/_media/RhythmCalibration.mp4' | relative_url }}" type="video/mp4">
</video>
</div>

Several questions did arise when designing the calibration feature: Should each player have individual calibration settings in the case that not everyone is using the same output device or to accommodate individual reaction times? Or should we simply take the average of all players calibrating at once? Or should only player one be responsible for calibration? The first option is the most attractive in terms of making the game accessible. However, the idea with the calibration is that in the game when the player hits the button on the visual cue, it lines up exactly when they would hit the button on the audio cue. Simply put, it is impossible to accommodate for all players a perfect calibration. That is why I decided to go with the option of having the calibration be the average of all the players who participate in the calibration. After all, it is a collaborative effort and the main idea behind the mechanic is for everyone to be in sync, so that must be honored. Even with calibration, being in sync takes some skill. Even with calibration, the latency of input feedback cannot be accounted for. Calibration does not solve every issue, so we must accept the reality of hardware device limitations and human inaccuracy.
