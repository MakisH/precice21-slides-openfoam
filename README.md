# My preCICE Workshop 2021 presentation slides (2)

- Title: The OpenFOAM-preCICE adapter
- Speaker: Gerasimos Chourdakis, Technical University of Munich
- Event: [preCICE Workshop 2021](https://www.precice.org/precice-workshop-2021.html), online
- Date: February 24, 2021

See also the slides of my talk "[The preCICE community](https://github.com/MakisH/precice21-slides-community)"

## Build

Follow the instructions on [reveal.js](https://revealjs.com/installation/), or just install Node.js 10.0.0 or later and do:

```bash
npm install
npm start
```

and go to [localhost:8000](http://localhost:8000/) to see the slides.

## Convert to PDF

See section "[Export to PDF](https://revealjs.com/pdf-export/)" in the reveal.js documentation.

[Decktape](https://github.com/astefanutti/decktape) does a marvelous job converting this presentation to PDF. Get the Docker image (see Decktape README) and run (for localhost):

```bash
docker run --rm -t --net=host -v "$(pwd)":/slides astefanutti/decktape generic --key=" " -p 2000 -s 1920x1440 http://localhost:8000 slides.pdf
```

## License & more

- License: [CreativeCommons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/)
- Based on [reveal.js](https://github.com/hakimel/reveal.js). Template based on the "White" template by Hakim El Hattab.
- The TUM logo is part of the corporate identity of the Technical University of Munich.