# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/main/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm create svelte@latest

# create a new project in my-app
npm create svelte@latest my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.


# Notes

## object
http://fabricjs.com/docs/fabric.Object.html#excludeFromExport
excludeFromExport - add this propery to guide lines, if drawing any

- opacity  - http://fabricjs.com/docs/fabric.Object.html#opacity


oCoords
Describe object's corner position in canvas element coordinates. properties are depending on control keys and padding the main controls. each property is an object with x, y and corner. The `corner` property contains in a similar manner the 4 points of the interactive area of the corner. The coordinates depends from the controls positionHandler and are used to draw and locate control



oCoords


alinment system
https://chatgpt.com/share/67011ba7-2830-8003-b2a5-0f08fe9f67e6

snappy rect
https://github.com/dinesh-rawat-dev/fabricjs-prodeasy-snappy-rect?ref=hackernoon.com
https://hackernoon.com/mastering-object-snapping-in-fabricjs-introducing-the-snappyrect-class


in the future i need to create this library

but for now, i can get awy with scaling everything to width aand hight
