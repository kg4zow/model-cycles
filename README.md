# Information about the Elektron Model:Cycles

This repo contains information I've figured out while reverse-engineering the format of the `.mcpst` files created by the [Elektron Model:Cycles](https://www.elektron.se/products/modelcycles/). These files contain "Presets", which are collections of settings which are used to configure what kinds of sound the Model:Cycles FM synthesizers produce.

The `docs/` directory contains the actual documentation I've written.

The `scripts/` directory contains a couple of scripts I wrote while figuring out the file format.

* **`mcpst-dump`** is the script I had originally envisioned, which reads an `.mcpst` file and prints the values of the various parameters stored in the Preset it contains.

* **`mcpst-compare`** is a script I wrote to make it easier for me to compare two or more `.mcpst` files. It reads the files into memory and shows hex dumps with the differences highlighted. This script made the whole reverse-engineering process SO much faster and easier.
