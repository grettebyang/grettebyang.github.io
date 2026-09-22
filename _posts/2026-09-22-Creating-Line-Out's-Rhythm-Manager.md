## Creating Line Out's Rhythm Manager

My first time programming rhythm mechanics for a game was a humbling experience and took several iterations to get it working as intended. It's still not perfect, and there are many other ways it could be done over, but I learned a lot and look forward to my next project where I can make an even better rhythm manager.


### Making custom soundtracks
For the rhythm manager to keep the beat, it needs information directly from the music, so I created a Soundtrack class which holds the BPM, how many bars in the track, the current playback position, and other variables. In the game, when a rhythm event is triggered, the sequence you must perform depends on which part of the song is playing. This means that the soundtrack must also have a list of the sequences as well as how many bars each one lasts for.
What's more is when I want to make a song that has varying time signature changes, then the soundtrack must also have a list of these time signatures and how many bars each one lasts for.


### Calibrating the rhythm
Because some audio output devices have a considerable amount of delay, it was crucial that I implemented a way for players to be able to calibrate the timing of the rhythm. 

Here's a clipping from the Soundtrack class which updates the playback position while taking into account the audio calibration settings:
```cs
  _curPlaybackPosition = GetPlaybackPosition() + (float)AudioServer.GetTimeSinceLastMix();
  _calibratedPlaybackPosition = ((_curPlaybackPosition + _audioCalibrationOffset) + (float)Stream.GetLength()) % (float)Stream.GetLength();
  _metDelta = _calibratedPlaybackPosition - _metDelta;
```
I actually implemented two modes of calibration: one for audio latency, and one for video latency, to calibrate the visual cues as well. The calibration not only takes into account device latency, but also player reaction timing. Rhythm calibration settings are a must-have for any rhythm game.

