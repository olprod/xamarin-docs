---
title: "Xamarin.Essentials Text-to-Speech"
description: "The TextToSpeech class enables an application utilize the built in text-to-speech engines to speak back text from the device and also to query available languages that the engine can support."
ms.assetid: AEEF03AE-A047-4DF0-B0E8-CC8D9A7B8351
author: jamesmontemagno
ms.author: jamont
ms.date: 05/04/2018
---
# Xamarin.Essentials Text-to-Speech

![Pre-release NuGet](~/media/shared/pre-release.png)

The **TextToSpeech** class enables an application to utilize the built in text-to-speech engines to speak back text from the device and also to query available languages that the engine can support.

## Using Text-to-Speech

Add a reference to Xamarin.Essentials in your class:

```csharp
using Xamarin.Essentials;
```

The Text-to-Speech functionality works by calling the `SpeakAsync` method with text and optional parameters and returns after the utterance has finished. 

```csharp
public async Task SpeakNowDefaultSettings()
{
    await TextToSpeech.SpeakAsync("Hello World");

    // This method will block until utterance finishes.
}

public void SpeakNowDefaultSettings2()
{
    TextToSpeech.SpeakAsync("Hello World").ContinueWith((t) => 
    {
        // Logic that will run after utterance finishes.

    }, TaskScheduler.FromCurrentSynchronizationContext());
}
```

This method takes in an optional CancellationToken to stop the utterance once it starts. 
```csharp
CancellationTokenSource cts;
public async Task SpeakNowDefaultSettings()
{
    cts = new CancellationTokenSource();
    await TextToSpeech.SpeakAsync("Hello World", cancelToken: cts.Token);

    // This method will block until utterance finishes.
}

public void CancelSpeech()
{
    if (cts?.IsCancellationRequested ?? false)
        return;

    cts.Cancel();
}
```

Text-to-Speech will automatically queue speech requests from the same thread. 

```csharp
bool isBusy = false;
public void SpeakMultiple()
{
    isBusy = true;
    Task.Run(async () =>
    {
        await TextToSpeech.SpeakAsync("Hello World 1");
        await TextToSpeech.SpeakAsync("Hello World 2");
        await TextToSpeech.SpeakAsync("Hello World 3");
        isBusy = false;
    });

    // or you can query multiple without a Task:
    Task.WhenAll(
        TextToSpeech.SpeakAsync("Hello World 1"),
        TextToSpeech.SpeakAsync("Hello World 2"),
        TextToSpeech.SpeakAsync("Hello World 3"))
        .ContinueWith((t) => { isBusy = false; }, TaskScheduler.FromCurrentSynchronizationContext());
}
```

### Speech Settings

For more control over how the audio is spoken back with `SpeakSettings` that allows setting the Volume, Pitch, and Locale.

```csharp
public async Task SpeakNow()
{
    var settings = new SpeakSettings()
        {
            Volume = .75,
            Pitch = 1.0
        };

    await TextToSpeech.SpeakAsync("Hello World", settings);
}
```

The following are supported values for these parameters:

| Parameter | Minimum | Maximum |
| --- | :---: | :---: |
| Pitch | 0 | 2.0 |
| Volume | 0 | 1.0 |

### Speech Locales

Each platform offers locales to speak back text in multiple languages and accents. Each platform has different codes and ways of specifying this, which is why Essentials provides a cross-platform `Locale` class and a way to query them with `GetLocalesAsync`.

```csharp
public async Task SpeakNow()
{
    var locales = await TextToSpeech.GetLocalesAsync();

    // Grab the first locale
    var locale = locales.FirstOrDefault();

    var settings = new SpeakSettings()
        {
            Volume = .75,
            Pitch = 1.0,
            Locale = locale
        };

    await TextToSpeech.SpeakAsync("Hello World", settings);
}
```

## Limitations

- Utterance queue is not guaranteed if called across multiple threads.
- Background audio playback is not officially supported.

## API

- [TextToSpeech source code](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/TextToSpeech)
- [TextToSpeech API documentation](xref:Xamarin.Essentials.TextToSpeech)
