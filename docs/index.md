# Cute Asset Pipeline

* [What is it?](#what-is-it)
* [Supported Operating Systems](#supported-operating-systems)
* [Getting Started](#getting-started)
  * [Requirements](#requirements)
  * [Asset Database](#asset-database)
    * [Asset Database Schema](#asset-database-schema)
  * [Output File](#output-file)
  * [Config](#config)
    * [Config File](#config-file)
    * [Command Line Options](#command-line-options)
* [Usage](#usage)

## What is it?

Cute asset pipeline is an easy to use and automate way to change your PSD files directly to PNGs and a texture atlas.
It additionally adds a way to package your assets and metadata about your assets with the same tool. Something other texture packing programs don't easily support.

## Supported Operating Systems

Cute Asset Pipeline should work on windows and MacOS.

## Getting started

### Requirements
Currently Cute Asset Pipeline requires at you to have [Image Magick](https://imagemagick.org/script/download.php) installed. Later versions should remove this requirement. Make sure image magick is added to your path. This should already be the case on OSX. If you cannot put Image Magick in your path you can specify the path to the `convert` program in the [config file](#config)

### Asset Database
Cute Asset Pipeline uses an sqlite database to track assets. You can intialize a new one by running `exporter init-db <db_name>` this will create a file in your current directory with the name you passed as `db_name`. Cute Asset Pipeline currently doesn't have a nice frontend for accessing you will either need to know how you write your own queries from your command line or download a GUI options like [Sqlite Browser](https://sqlitebrowser.org/dl/). 
#### Asset Database Schema
The schema is fairly simple with a few core concepts.

* `raw_assets` This is your raw PSD files that you want to convert to PSD. 
PSD files can have multiple layers corresponding to multiple images. Cute Asset Pipeline currently does not support merging layers or layer groups to composite a single PNG. It is 1 layer to 1 PSD. However, Cute Asset Pipeline does support groups and nested groups. 
  * `path` this will be the file path and name for your PSD. It does not have to be absolute, you will be able to set up an asset path directory config option in the [config file](#config). examples might be `characters/player.psd` or simply `elf.psd`.
  * `group` And optional value to name this group of layer(s) in this PSD. Does not have to be unique. You might want all your player assets to be have a group name of `player` for instance. This value will be set as `groupName` for every asset in the group in the [output file](#output-file).
* `game_asset` This has the data for the actual asset definitions you want to export. If you had multiple layers from your raw asset you wanted to export you would define each layer here.
  * `id` auto assigned id you can refer to this asset with.
  * `name` what you want to name the asset. Can be any value you choose.
  * `path_in_parent` the group/layer name to identify the asset layer in the parent `raw_asset`. If the layer for your player facing forward is in the group `front` and has the layer name `idle` the `path_in_parent` would be `front/idle` you can go as many nested groups in your PSD as you want. eg `front/walk/walk_1`.
  * `raw_asset_id` the id of the corresponding `raw_asset` that contains this asset.
  * `active` set this to 0 or 1 depending if you want to the asset to be exported.
  * `animation_id` animation id for linking animation info. Not required.
  * `frame` the frame number in the animation. Not required.
  * `resolution` the asset can be scaled from the size of the layer. Valid formats"
    * `x<height>` Set the height of the resulting image. Will keep aspect ratio. Example: `x500` will set the height to 500px
    * `<width>x` Set the width of the resulting image. Will keep aspect ratio. Example `500x` will set the width to 500px
    * `<value>%` Scale the image by percent. Will keep aspect ratio. Example `75%` an 100px by 100px layer will become `75px by 75px`
* `animation` defines an animation. Very simple at this point. 
  * `name` name for this animation. Will group `game_assets` in the [output_file](#output-file) by frame number.
* `tag` freeform tags 
  * `name` Name for the tag. Can be any value.
  * `game_asset_id` `game_asset` to apply the tag to.
  
### Output File
The result of running the exporter will result in two files `out.png` and `out.json`. `out.png` is an image atlas containg all the defined `game_assets`. `out.json` contains definitions for all the `game_asset`s and their position and size in the `out.png`. The name for the output files and path the output file is configurable in the [Config](#config-file)

#### JSON file format
JSON file contains all the information related to exported assets and animations.
Example file:

```json
{
  "frames": [
    {
      "name": "exmaple",
      "tags": ["some", "tags"],
      "groupName": "parent",
      "x": 10,
      "y": 20,
      "width": 200,
      "height": 300,
      "animationId": 1,
      "frameNumber": 1,
      "animationName": "animation"
    },
    {
      "name": "exmaple",
      "tags": ["some", "tags"],
      "groupName": "parent",
      "x": 10,
      "y": 20,
      "width": 200,
      "height": 300,
      "animationId": null,
      "frameNumber": -1,
      "animationName": null
    }
  ],
  "animations": [
    {
      "name": "animation",
      "assets": [1]
    }
  ]
}
```

### Config

You can configure the settings for exporting with either command line options or a config file. 
Config values:
* `output-dir`
  * Directory the exporter should output the output png and json to.
  * defaults to the current directory
* `output-name`
  * name use use for the .json and .png outputted files in the `output-dir` .json and .png will be added automatically so don't add a file extension
  * default to `out` (thereby creating `out.png` and `out.json`)
* `cache-dir`
  * the exporter will cache the intermediate pngs to a directory so subsequent runs will be faster.
  * default to `cache`
* `asset-dir`
  * directory to prepend to the `raw_asset.path` when trying to load a PSD. Eg if `asset-dir` is `assets` and the `raw_asset.path` is `player/cool.psd` exporter will look for the psd at `assets/player/cool.psd`
  * default to the current directory
* `padding` 
  * padding, in pixels, around each exported asset in the final texture atlas. Useful for making sure pixels from one asset don't blend into another.
  * default to 4.

#### Config File
By default Cute Asset Pipeline will look for a file name `cute.ini` in the directory it is being run from you can specify the name and path by passing `--config <path to config>`.

All values are optional.
Format of the config file.

```
output-dir=output
output-name=files
asset-dir=that
cache-dir=something
padding=8
```

#### Command line options
The same options in the config file are available to be passed from the command line. Prefix the option with `--` for the command line eg `output-dir=this` in the config file would be `--output-dir this` on the command line. Command line options override config file options.


## Usage

These commands are run on the command line or terminal.
### Creating intial asset db
`cute_asset_pipeline init <db_file_name>`

example: `cute_asset_pipeline init cute.ini` will create a db file in the current directory
### Exporting assets
`cute_asset_pipeline export`

You can also pass [command line options](#command-line-options) here or specify `--config` to specify a different path/name to the default `cute.ini` in the current directory.

examples: 
`cute_asset_pipeline export --config assets/cute.ini`
`cute_asset_pipeline export --asset-dir assets`


