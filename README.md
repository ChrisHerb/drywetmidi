![DryWetMIDI Logo](https://github.com/melanchall/drywetmidi/blob/develop/Images/dwm-logo.png)

DryWetMIDI is the .NET library to work with MIDI files. Visit [Wiki](https://github.com/melanchall/drymidi/wiki) to learn how to use the DryWetMIDI. You can get the latest version via NuGet:

[![NuGet](https://img.shields.io/nuget/v/Melanchall.DryWetMidi.svg?style=flat-square)](https://www.nuget.org/packages/Melanchall.DryWetMidi/)

DryWetMIDI was tested on 130,000 files taken from [here](https://www.reddit.com/r/WeAreTheMusicMakers/comments/3ajwe4/the_largest_midi_collection_on_the_internet/). Thanks *midi-man* for this great collection.

## Features

With the DryWetMIDI you can:

* Read, write and create [Standard MIDI Files (SMF)](https://www.midi.org/specifications/category/smf-specifications). It is also possible to read [RMID](https://www.loc.gov/preservation/digital/formats/fdd/fdd000120.shtml) files where SMF wrapped to RIFF chunk.
* Finely adjust process of reading and writing. It allows, for example, to read corrupted files and repair them, or build MIDI file validators.
* Implement [custom meta events](https://github.com/melanchall/drywetmidi/wiki/Custom-meta-events) and [custom chunks](https://github.com/melanchall/drywetmidi/wiki/Custom-chunks) that can be written to and read from MIDI files.
* Easily catch specific error when reading or writing MIDI file since all possible errors in a MIDI file are presented as separate exception classes.

## Getting Started

Let's see some examples of what you can do with DryWetMIDI.

To [read a MIDI file](https://github.com/melanchall/drymidi/wiki/Reading-a-MIDI-file) you have to use ```Read``` static method of the ```MidiFile```:

```csharp
var midiFile = MidiFile.Read("Some Great Song.mid");
```

or, in more advanced form (visit [Reading settings](https://github.com/melanchall/drywetmidi/wiki/Reading-settings) page on Wiki to learn more about how to adjust process of reading)

```csharp
var midiFile = MidiFile.Read("Some Great Song.mid",
                             new ReadingSettings
                             {
                                 NoHeaderChunkPolicy = NoHeaderChunkPolicy.Abort,
                                 CustomChunkTypes = new ChunkTypesCollection
                                 {
                                     { typeof(MyCustomChunk), "Cstm" }
                                 }
                             });
```

To [write MIDI data to a file](https://github.com/melanchall/drymidi/wiki/Writing-a-MIDI-file) you have to use ```Write``` method of the ```MidiFile```:

```csharp
midiFile.Write("My Great Song.mid");
```

or, in more advanced form (visit [Writing settings](https://github.com/melanchall/drywetmidi/wiki/Writing-settings) page on Wiki to learn more about how to adjust process of writing)

```csharp
midiFile.Write("My Great Song.mid",
               true,
               MidiFileFormat.SingleTrack,
               new WritingSettings
               {
                   CompressionPolicy = CompressionPolicy.Default
               });
```

Of course you can create a MIDI file from scratch by creating an instance of the ```MidiFile``` and writing it:

```csharp
var midiFile = new MidiFile(
                   new TrackChunk(
                       new SetTempoEvent(500000)),
                   new TrackChunk(
                       new TextEvent("It's just single note track..."),
                       new NoteOnEvent((SevenBitNumber)60, (SevenBitNumber)45),
                       new NoteOffEvent((SevenBitNumber)60, (SevenBitNumber)0)
                       {
                           DeltaTime = 400
                       }));
midiFile.Write("My Future Great Song.mid");
```

If you want to speed up playing back a MIDI file by two times you can do it with this code:

```csharp                   
foreach (var trackChunk in midiFile.Chunks.OfType<TrackChunk>())
{
    foreach (var setTempoEvent in trackChunk.Events.OfType<SetTempoEvent>())
    {
        setTempoEvent.MicrosecondsPerBeat /= 2;
    }
}
```

Of course this code is simplified. In practice a MIDI file may not contain SetTempo event which means it has the default one (500,000 microseconds per beat).

Suppose you want to remove all C# notes from a MIDI file. It can be done with this code:

```csharp
foreach (var trackChunk in midiFile.Chunks.OfType<TrackChunk>())
{
    trackChunk.Events.RemoveAll(e => (e as NoteOnEvent)?.GetNoteName() == NoteName.CSharp ||
                                     (e as NoteOffEvent)?.GetNoteName() == NoteName.CSharp);
}
```
