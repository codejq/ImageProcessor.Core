# ImageProcessor.Core

```text
 ╔══════════════════════════════════════════════════════════════╗
 ║                                                              ║
 ║   ██╗███╗   ███╗ █████╗  ██████╗ ███████╗                   ║
 ║   ██║████╗ ████║██╔══██╗██╔════╝ ██╔════╝                   ║
 ║   ██║██╔████╔██║███████║██║  ███╗█████╗                     ║
 ║   ██║██║╚██╔╝██║██╔══██║██║   ██║██╔══╝                     ║
 ║   ██║██║ ╚═╝ ██║██║  ██║╚██████╔╝███████╗                   ║
 ║   ╚═╝╚═╝     ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝                   ║
 ║                                                              ║
 ║        P R O C E S S O R  ·  C O R E                        ║
 ║                                                              ║
 ║   Fluent image processing — built on .NET, runs anywhere    ║
 ╚══════════════════════════════════════════════════════════════╝
```

[![NuGet](https://img.shields.io/nuget/v/ImageProcessor.Core.svg)](https://www.nuget.org/packages/ImageProcessor.Core/)
[![Build](https://github.com/codejq/ImageProcessor.Core/actions/workflows/build.yml/badge.svg)](https://github.com/codejq/ImageProcessor.Core/actions/workflows/build.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![.NET](https://img.shields.io/badge/.NET-Standard%202.0%20%7C%208%20%7C%209-purple)](https://dotnet.microsoft.com)
[![Docs](https://img.shields.io/badge/docs-GitHub%20Pages-blueviolet)](https://codejq.github.io/ImageProcessor.Core/)

---

## Why do you need this?

```text
  You have an image.                    You want this:
  ┌──────────────┐                      ┌──────────────┐
  │  raw photo   │   ─────────────►     │  processed   │
  │  ugly size   │   one fluent chain   │  watermarked │
  │  no watermark│                      │  resized     │
  │  wrong format│                      │  PNG / WEBP  │
  └──────────────┘                      └──────────────┘

  Standard .NET gives you this:
    var bmp = new Bitmap(file);
    var g   = Graphics.FromImage(bmp);
    g.DrawImage(...);          // resize — 20 lines
    g.DrawString(...);         // watermark — 15 lines
    bmp.Save(file, codec);     // format — 10 more lines

  ImageProcessor.Core gives you this:
    new ImageFactory()
        .Load(file)
        .Resize(new ResizeLayer(800, 600))
        .Watermark(new TextLayer { Text = "© Me" })
        .Format(new PngFormat())
        .Save(output);         // done. 6 lines.
```

> **One library. Fluent API. No ceremony.**
> Stop wiring `Graphics`, `ImageCodecInfo`, and `EncoderParameters` by hand.
> Every operation is a single chained call — readable, testable, and composable.

---

## What it can do

```text
  ┌─────────────────────────────────────────────────────────┐
  │                   ImageFactory                          │
  │                                                         │
  │  Load ──┬── Resize ──── Crop ──── Rotate ── Save       │
  │         │                                               │
  │         ├── Filters ─┬─ GaussianBlur                   │
  │         │            ├─ GaussianSharpen                 │
  │         │            ├─ DetectEdges                     │
  │         │            ├─ Gray / Binary / Halftone        │
  │         │            └─ Tint / Vignette / Hue           │
  │         │                                               │
  │         ├── Adjust ──┬─ Brightness / Contrast          │
  │         │            ├─ Saturation / Gamma              │
  │         │            ├─ Alpha / ReplaceColor            │
  │         │            └─ Resolution / Quality            │
  │         │                                               │
  │         ├── Compose ─┬─ Watermark (text)               │
  │         │            ├─ Overlay (image on image)        │
  │         │            ├─ Mask                            │
  │         │            └─ Background / BackgroundColor    │
  │         │                                               │
  │         ├── Special ─┬─ Mosaic (pixelated region)      │
  │         │            ├─ Pixelate                        │
  │         │            ├─ RoundedCorners                  │
  │         │            ├─ EntropyCrop (smart crop)        │
  │         │            ├─ ClearNoise / ClearBackground    │
  │         │            └─ Flip / AutoRotate / Scale       │
  │         │                                               │
  │         └── Format ──┬─ JPEG / PNG / GIF / BMP         │
  │                      └─ (preserves animation in GIF)   │
  └─────────────────────────────────────────────────────────┘
```

---

## Install

```bash
dotnet add package ImageProcessor.Core
```

Or via Package Manager:

```powershell
Install-Package ImageProcessor.Core
```

---

## Quick start

```csharp
using ImageProcessor;
using ImageProcessor.Imaging;
using ImageProcessor.Imaging.Formats;

// Resize + sharpen + save as PNG
using var factory = new ImageFactory();
factory.Load("input.jpg")
       .Resize(new ResizeLayer(1280, 720))
       .GaussianSharpen(5)
       .Format(new PngFormat())
       .Save("output.png");
```

```csharp
// Mosaic (pixelate) a region — great for censoring faces / text
factory.Load("screenshot.png")
       .Mosaic(new MosaicLayer(x: 100, y: 50, width: 200, height: 80,
                               size: new Size(10, 10)))
       .Save("censored.png");
```

```csharp
// Add a watermark
factory.Load("photo.jpg")
       .Watermark(new TextLayer
       {
           Text     = "© My Company",
           FontSize = 24,
           Position = new Point(20, 20),
           Opacity  = 80
       })
       .Save("photo_watermarked.jpg");
```

```csharp
// Chain multiple operations
factory.Load("raw.gif")
       .AutoRotate()
       .Resize(new ResizeLayer(800, 600, ResizeMode.Crop))
       .Brightness(10)
       .Contrast(5)
       .Quality(85)
       .Save("final.gif");  // GIF animation preserved
```

---

## Mosaic / censoring in detail

```text
  Original image           After Mosaic(x:450, y:280, w:140, h:50)
  ┌─────────────────┐      ┌─────────────────┐
  │                 │      │                 │
  │   [face here]   │      │   [██████████]  │  ← pixelated block
  │                 │      │                 │
  └─────────────────┘      └─────────────────┘
```

```csharp
factory.Load("photo.gif")
       .Mosaic(new MosaicLayer(450, 280, 140, 50, new Size(10, 10)))
       .Save("photo_censored.gif");
```

---

## Pipeline architecture

```text
              ┌──────────┐
   file/stream│          │
  ──────────► │  .Load() │
              └────┬─────┘
                   │  Image + Format detected
                   ▼
          ┌────────────────┐
          │ Processor chain│  ◄── each .Method() enqueues a processor
          │  [ P1, P2, P3 ]│
          └────────┬───────┘
                   │  applied in order
                   ▼
              ┌──────────┐
              │ .Save()  │──────► file / stream
              └──────────┘
```

Each processor implements `IGraphicsProcessor` — you can also **add your own**:

```csharp
public class MyProcessor : IGraphicsProcessor
{
    public Image ProcessImage(ImageFactory factory)
    {
        // manipulate factory.Image
        return factory.Image;
    }
}

factory.Load("img.png")
       .ApplyProcessor(new MyProcessor())
       .Save("out.png");
```

---

## Target frameworks

| Framework | Support | Notes |
| --- | --- | --- |
| .NET Standard 2.0 | ✅ | .NET Framework 4.6.1+, .NET Core 2.0+ |
| .NET 8 (Windows) | ✅ | Uses native GDI+ via System.Drawing.Common |
| .NET 9 (Windows) | ✅ | Uses native GDI+ via System.Drawing.Common |

> `System.Drawing.Common` is Windows-only on .NET 6+. This library targets `net8.0-windows`
> and `net9.0-windows` for modern .NET to make that constraint explicit and safe.

---

## Contributing

PRs and issues welcome. Please open an issue before submitting large changes.

---

## License

[Apache License 2.0](LICENSE) — originally by
[James Jackson-South](https://github.com/JimBobSquarePants/ImageProcessor),
extended here with additional processors (Mosaic, ClearNoise, Binary, etc.).
