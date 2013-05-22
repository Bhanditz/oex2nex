# Oex2nex Convertor

The oex2nex convertor converts an Opera oex extension into an Opera 15 nex extension, using the [oex-shim](I assume this will be on GitHub too?) library. Extension authors can use this tool as a stopgap solution to bring existing content to the [new Opera extension architecture](link to dev.opera article explaining this).

## What does this actually do?

oex2nex parses the `config.xml` file of an `oex` extension and creates a `manifest.json` file containing the relevant metadata, API permissions, and references to assets. It also parses extension code to make scope fixes, modify ASTs and generally perform all other black magic required to create a compatibility layer between the two extension models.

## Usage

You can run oex2nex as a simple command-line utility, or install it as a package.

### Command-line

`convertor.py [-h] [-s KEY] [-x] [-d] [-f] [in_file] [out_file]`

```
positional arguments:
  in_file            Path to an .oex file or a directory where its extracted
                     contents are available
  out_file           Output file path (a .nex file or a directory)

optional arguments:
  -h, --help         show this help message and exit
  -s KEY, --key KEY  Sign the nex package with the provided key (PEM) file.
                     The signed package is named <file>.signed.nex.
  -x, --outdir       Create or use a directory for output
  -d, --debug        Debug mode; quite verbose
  -f, --fetch        Fetch the latest oex_shim scripts and put them in
                     oex_shim directory.
```

For example, to convert an Opera `oex` extension dino-comics.oex into a `nex` compatible with Opera 14, but output the exension's contents as a directory (useful for tweaking things):

```
$ python oex2nex/convertor.py -xd path/to/dino-comics.oex path/to/put/dino-comics
```

### Installing as a package

```
$ python setup.py install
```

## Tests

Currently we have a handful of tests in oex2nex/tests. You can run them like so:

```
$ python oex2nex/test.py
```
Better test coverage is always a good thing, so feel free to contribute back tests with any improvements.

## Known Issues

### Unimplemented APIs

The following OEX and Opera APIs aren't supported by the oex-shim:

* `window.opera` [User JS methods, events, or properties](http://www.opera.com/docs/userjs/specs/), i.e., `opera.addEventListener`, `opera.defineMagicVariable`, etc.

### Icons

It's possible that the icons that end up in your NEX package aren't the ideal resolution. Here's roughly how icons are selected when parsing your config.xml and OEX container:

* If there is an `<icon>` element with a `width` attribute of 16, 48, or 128, grab those.
* IF NOT, if there is the number 16, 48, or 128 somewhere in the file name, e.g., `pretty_icon_16px.png`, grab those.
* IF NOT, grab the only icon and use that as the 128 icon.

So if you notice that the icons aren't quite right in the generated NEX container, you should modify the manifest.json to point to the correct assetes.

### JavaScript Parsing

We use [Slimit](https://github.com/rspivak/slimit) to parse OEX JavaScript files. While the vast majority of extensions that we've converted work without an issue—it's possible that there is [some](https://github.com/rspivak/slimit/issues/42) [syntax](https://github.com/rspivak/slimit/pull/45) or [construct](https://github.com/rspivak/slimit/pull/44) that Slimit fails to handle correctly. In these cases, you can open an issue on this project and we can investigate further.