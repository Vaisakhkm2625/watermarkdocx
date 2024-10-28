# Watermark docx images

> WIP.. Proof of concept... 
> need to rewrite in svelte5 and need to properly splitting components

This application can batch edit all the pictures in a docx file, uploaded by user

## Why?

There isn’t a simple application that can do this; at least, I wasn’t able to find one. Even enterprise offerings like [docxtemplater](https://docxtemplater.com) require a custom extension to support this feature, even with a subscription.

Our university requires assignments to be submitted with roll numbers as watermark in each images of documents, to avoid plagiarisms - which for a 50 page doc group project with 10 group members, can take few hours to watermark more than 500 images manually



## Developing

```bash
git clone git@github.com:Vaisakhkm2625/watermarkdocx.git
npm install
npm run dev # opens in host mode at localhost:8080
```

## Building

To create a production version of your app and deploy to github pages:

(create a gh-pages branch in github and point gh pages to that branch)
```bash
npm run build
npm run deploy
```

# Notes

## object
http://fabricjs.com/docs/fabric.Object.html#excludeFromExport
excludeFromExport - add this propery to guide lines, if drawing any

- opacity  - http://fabricjs.com/docs/fabric.Object.html#opacity


oCoords
Describe object's corner position in canvas element coordinates. properties are depending on control keys and padding the main controls. each property is an object with x, y and corner. The `corner` property contains in a similar manner the 4 points of the interactive area of the corner. The coordinates depends from the controls positionHandler and are used to draw and locate control



oCoords


alignment system
https://chatgpt.com/share/67011ba7-2830-8003-b2a5-0f08fe9f67e6

snappy rect
https://github.com/dinesh-rawat-dev/fabricjs-prodeasy-snappy-rect?ref=hackernoon.com
https://hackernoon.com/mastering-object-snapping-in-fabricjs-introducing-the-snappyrect-class


in the future i need to create this library

but for now, i can get away with scaling everything to width and hight
